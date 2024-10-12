---
layout: post
title: "REST API개발:클라이언트측 파일 다운로드 문제 해결과정"
date: 2024-10-12 12:41:00 +09:00
categories: Django
tags: Django DRF Javascript-error() 
---

**REST API 개발 일지: 파일 다운로드 문제 해결 과정**

이번 개발 일지에서는 REST API를 통해 클라이언트와 서버 간에 MP3 파일을 주고받는 과정에서 겪었던 문제와 그 해결 과정을 기록하려고 한다. 이번 경험을 통해 API와 쿼리 파라미터의 사용, 파일명 인코딩, 그리고 사용자 피드백을 위한 UI 구현에 대해 많은 것을 배웠다.

### 1. REST API 서버 구축 및 테스트 시작

먼저, REST 서버를 통해 비디오 URL을 받아 오디오 파일(MP3)을 생성하는 **`DownloadAudioView`**와 생성된 파일을 클라이언트에서 다운로드할 수 있는 **`FileDownloadView`**를 만들었다. Django REST Framework를 이용해 간단히 **GET**과 **POST** 요청을 처리하는 클래스를 작성했다.

처음 테스트에서는 클라이언트 측에서 비디오 URL을 `POST`로 서버에 전송하면, 서버는 해당 URL에서 오디오를 추출하여 MP3 파일로 저장한 후, 다운로드 링크를 클라이언트에 반환하는 구조였다.

### 2. 발생한 이슈

#### 2.1 MP3 자동 재생 문제

테스트 중 클라이언트에서 다운로드 링크를 통해 MP3 파일을 받을 때, 파일이 다운로드되지 않고 브라우저에서 **자동으로 재생되는 문제**가 발생했다. 이 문제는 브라우저가 파일을 스트리밍 콘텐츠로 인식하여 발생한 것으로, **`Content-Disposition`** 헤더를 통해 명시적으로 **파일을 첨부 파일로 처리**하도록 설정하여 해결할 수 있었다.

이를 위해 **`Content-Type`**을 **`application/octet-stream`**으로 설정하고, **`Content-Disposition`** 헤더에 **`attachment`**를 명시하여 브라우저가 파일을 다운로드하게끔 처리했다.

#### 2.2 파일 다운로드 시 잘못된 파일명

테스트 중 클라이언트에서 파일을 다운로드할 때 **파일명이 의도한 대로 나타나지 않고, 기본적으로 "다운로드.mp3"라는 이름으로 표시되는 문제**를 발견했다. 또한, 파일명이 한글인 경우 브라우저에서 파일명이 깨지거나 기본값으로 바뀌는 현상이 발생했다.

문제의 원인을 분석한 결과, **`Content-Disposition`** 헤더의 설정에 인코딩 문제가 있다는 것을 알게 되었다. 특히, 한글 파일명과 같은 **비 ASCII 문자**가 제대로 처리되지 않아 발생한 문제였다.

### 3. 문제 해결

#### 3.1 MP3 자동 재생 문제 해결

MP3 자동 재생 문제는 **`Content-Disposition`** 헤더를 수정하여 해결했다. 브라우저가 파일을 첨부 파일로 인식하도록 하기 위해 **`attachment`**를 명시하고, **`Content-Type`**을 **`application/octet-stream`**으로 설정했다.

#### 3.2 파일명 인코딩 문제 해결

파일명 인코딩 문제를 해결하기 위해 **파일명 인코딩**을 설정하는 방식으로 코드를 수정했다. HTTP 규격에 따라 **`filename*`**과 **UTF-8 인코딩**을 사용하여 파일명을 안전하게 전달하도록 했다.

수정된 코드는 다음과 같다:

```python
from django.http import FileResponse, Http404
import urllib.parse

class FileDownloadView(View):
    def get(self, request, *args, **kwargs):
        file_name = request.GET.get('file_name')
        if not file_name:
            raise Http404("File name is required")
        
        file_path = os.path.join(settings.MEDIA_ROOT, file_name)
        if not os.path.exists(file_path):
            raise Http404("File not found")

        # 파일명 인코딩 (특히 한글 처리)
        encoded_file_name = urllib.parse.quote(file_name)
        
        response = FileResponse(open(file_path, 'rb'))
        response['Content-Disposition'] = f"attachment; filename*=UTF-8''{encoded_file_name}"
        response['Content-Type'] = 'application/octet-stream'
        return response
```

이 코드를 통해 **모든 브라우저에서 한글 파일명도 제대로 인식**하고, 클라이언트가 파일을 다운로드할 때 의도한 파일명으로 저장되도록 할 수 있었다.

### 4. 클라이언트 측 에러 메시지 처리 방식 개선

에러 메시지를 표시할 때, 처음에는 브라우저의 기본 팝업 기능인 **`alert()`**을 사용했다. 이 방식은 간단하게 메시지를 띄우는 데 유용하지만, 커스터마이징이 불가능하다는 한계가 있다. 이후 더 세련된 사용자 경험을 위해 **Bootstrap 모달**을 사용해 에러 메시지를 표시하는 방식으로 개선했다.

```javascript
$('#downloadButton').click(function() {
    var videoUrl = $('#videoUrl').val();
    if (!videoUrl) {
        alert('Please enter a video URL');
        return;
    }

    $.ajax({
        url: 'https://rest.mikihands.com/api/downloader/download-audio/',
        type: 'POST',
        contentType: 'application/json',
        data: JSON.stringify({ 'video_url': videoUrl }),
        success: function(response) {
            if (response.download_url) {
                window.location.href = response.download_url;
            } else {
                $('#errorMessage').text('Failed to retrieve download link');
                $('#errorModal').modal('show');
            }
        },
        error: function(xhr, status, error) {
            $('#errorMessage').text('Error: ' + xhr.responseText);
            $('#errorModal').modal('show');
        }
    });
});
```

이렇게 하여 사용자가 에러를 보다 **명확하게 인식**할 수 있고, **더 나은 UI/UX**를 제공할 수 있게 되었다.

### 5. 배운 점

- **쿼리 파라미터를 사용한 API 통신**: 서버 내부에서 메서드 간에 변수 값을 전달할 때 쿼리 파라미터를 사용하는 방식은 Django의 전통적인 방식과는 달라 신선하게 다가왔다.
- **메서드의 액션 추가 인자 설정**: 동일한 클래스에서 여러 가지 요청 메서드를 효과적으로 처리하기 위해 **액션 추가 인자**를 설정하는 방법을 배우게 되었고, 이를 통해 코드의 가독성과 유지보수성을 높일 수 있었다. 이를 구현하기 위해 `urlpatterns`에서 각 메서드에 따라 URL 패턴을 설정하고, 메서드 내에서 인자를 처리하는 방법을 적용할 수 있었다.

예를 들어, `urlpatterns`에서 `action` 인자를 사용하는 방식으로 요청을 구분할 수 있다:

```python
from django.urls import path
from . import views

urlpatterns = [
    path('audio/', views.DownloadAudioView.as_view(), {'action': 'download'}, name='download-audio'),
    path('audio/', views.DownloadAudioView.as_view(), {'action': 'get'}, name='get-audio'),
]
```

이렇게 URL 패턴을 설정하고, **클래스 뷰**에서 `kwargs`로 전달된 `action` 인자를 활용해 메서드를 구분할 수 있다. 각 메서드는 다음과 같이 구현할 수 있다:

```python
from rest_framework.views import APIView
from rest_framework.response import Response

class DownloadAudioView(APIView):
    def post(self, request, *args, **kwargs):
        action = kwargs.get('action')
        if action == 'download':
            video_url = request.data.get('video_url')
            if not video_url:
                return Response({'error': 'No video URL provided'}, status=400)
            # 비디오 URL을 처리하여 오디오 파일 생성
            # ...
            return Response({'download_url': 'generated_download_link'})

    def get(self, request, *args, **kwargs):
        action = kwargs.get('action')
        if action == 'get':
            file_name = request.GET.get('file_name')
            if not file_name:
                return Response({'error': 'File name is required'}, status=400)
            # 파일명에 따른 파일 처리
            # ...
            return Response({'message': 'File retrieved successfully'})
```

이렇게 동일한 클래스 내에서 여러 액션을 처리하고, `urlpatterns`에서 각 메서드에 따른 `action` 인자를 설정함으로써 코드의 **재사용성**과 **유지보수성**을 높일 수 있었다.

- **파일명 인코딩 처리의 중요성**: 특히 비 ASCII 문자(한글, 특수문자 등)를 다룰 때는 파일명 인코딩을 제대로 처리하지 않으면 브라우저 호환성 문제가 발생할 수 있다는 것을 배웠다.
- **사용자 경험 향상**: `alert()`와 같은 기본적인 경고 창은 구현이 쉽지만, 더 나은 사용자 경험을 위해 Bootstrap 모달과 같은 커스터마이징 가능한 방법을 사용하는 것이 좋다.

📑 이번 일지에서 다룬 내용들은 나중에 비슷한 문제를 만났을 때 큰 도움이 될 것이다. 또한 ⚙️ API의 동작 원리와 🛠️ 클라이언트-🛠️ 서버 간의 데이터 전달 방식을 좀 더 깊이 이해할 수 있었던 좋은 경험이었다.