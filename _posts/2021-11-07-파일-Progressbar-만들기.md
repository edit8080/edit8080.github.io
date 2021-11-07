---
title: "클라이언트 상에서 동작하는 파일 Progressbar 만들기"
excerpt: "`<input type=file />` 이벤트 핸들링"

categories:
  - React
tags:
  - [File, input]

toc: true
toc_sticky: true

date: 2021-11-07
last_modified_at: 2021-11-07

published: true
---

## 1. 파일 Progress Bar 만들기

오늘 진행해볼 미니 프로젝트는 폼 상에서 파일을 업로드 할 때 시각적으로 보여줄 Progress Bar를 제작하는 것이다.
일반적인 구글링을 통해 살펴본 Progress Bar 제작 방법은 서버에 업로드하는 기능 위주에 치중해 있었다면,
지금 진행할 것은 `<input type=file />` 에서 파일을 로드할 때 서버에 request를 보내기 전 클라이언트 상에 올리는 것을 시각적으로 보여주기 위한
Progress Bar를 만드려고 한다.

## 2. 구현 계획

구현에서는 React를 활용한다. React로 파일을 올릴 수 있는 기초 폼을 만들고, 이벤트를 통해 파일의 업로드 상태를 확인한다.
이후, 동일한 파일이나 파일 크기 제한과 같은 유효성 검사 로직을 추가하여 완성도 있는 첨부파일 필드를 개발하는 것을 목적으로 한다.

1. 파일에 대한 기초 폼 제작
2. 파일 업로드 상태 확인하기
3. 파일 업로드에 대한 유효성 검사 로직 추가하기

## 3. 구현

### 3-1. 파일에 대한 기초 폼 제작

파일을 다룰 수 있는 가장 기본적인 폼을 구성하였다. 스타일은 `styled-components` 기법을 활용하였다.
`<input type="file">` 을 그대로 사용하면 <파일 선택> 이라는 문구가 옆에 보이는 것이 불편해 file input은 숨기고
버튼 클릭을 통해 file input을 트리거 하는 식으로 구성하였다.
이후 파일 업로드, 삭제에 대한 이벤트만 추가하여 업로드한 파일에 대해 정상적으로 렌더링 되는지 확인할 수 있도록 구성했다.

> `<input type="file" />` 을 통해 업로드한 파일들은 FileList 라는 형태로 저장되는데 이는 0부터 시작하는 이름을 가진 Object Array로 구성되어있다.
>
> ex) File {0: {File Info}, 1: {File Info}, ...}
>
> 따라서 `Array.forEach`로 접근하는 것이 불가능하고 우리가 잘 알고 있는 for 문으로 접근해야한다.

<iframe src="https://codesandbox.io/embed/pail-pom-guseonghagi-3pxfm?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="파일 폼 구성하기"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 3-2. 파일 업로드 상태 확인하기

파일의 업로드 상태를 확인하기 위해서는 `FileReader`라는 객체를 활용합니다.
`FileReader` 객체에서 지원하는 메서드와 이벤트 핸들러를 통해 파일의 업로드 상태를 손쉽게 접근할 수 있습니다.

> FileReader 객체에서 `readAsDataURL` 를 활용하여 `<input type="file">` 에서 업로드한 파일을 읽어들인다.
>
> 이벤트 핸들러는 파일 로딩 시작에 해당하는 `onload`, 파일 로딩 진행상태를 확인하는 `onprogress`, 파일 로딩 종료를 확인하는 `onloadend`를 활용한다.

파일을 업로드 하게 되면 업로드 상태에 따라 Progress bar가 변화되도록 구성해야합니다.
위 폼에서 구성한 내용에서는 `FileItem` 컴포넌트는 업로드할 파일이 추가되면 새롭게 렌더링되므로
`useEffect`를 통해 이벤트 트리거 시점을 결정할 수 있다.
이 프로젝트에서는 `useEffect`를 활용해 `useFileLoad`라는 이름의 커스텀 훅을 구성해서 파일의 로드 상태를 확인하도록 구현하였다.

```javascript

// useFileLoad.js
import { useCallback, useEffect, useState } from 'react'

const useFileLoad = (file) => {
  const [loadPercent, setLoadPercent] = useState(100)
  const [isLoading, setIsLoading] = useState(false)

  const handleEvent = useCallback((e) => {
    const { type, loaded, total } = e
    if (type === 'loadstart') {
      setLoadPercent(0)
      setIsLoading(true)
    } else if (type === 'progress') {
      setLoadPercent((loaded / total) * 100)
    } else if (type === 'loadend') {
      setIsLoading(false)
    }
  },[])

  const addEventToReader = useCallback((reader) => {
    reader.addEventListener('loadstart', handleEvent)
    reader.addEventListener('progress', handleEvent)
    reader.addEventListener('loadend', handleEvent)
  }, [])

  const readFile = useCallback(() => {
    const reader = new FileReader()

    addEventToReader(reader)
    reader.readAsDataURL(file)
  }, [file, addEventToReader])

  useEffect(() => {
    if (file instanceof File) readFile()
  }, [file, readFile])

  return [loadPercent, isLoading]
}
export default useFileLoad

```

<iframe src="https://codesandbox.io/embed/pail-eobrodeu-sangtae-hwaginhagi-s6v4x?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="파일 업로드 상태 확인하기"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### 3-3. 파일 업로드에 대한 유효성 검사 로직 추가하기

마지막으로 폼에서 빼놓을 수 없는 유효성 검사 로직을 추가한다.
유효성 검사는 동일한 파일명을 업로드하는 것을 제한하고 지정한 파일 제한 크기를 넘어가는지 확인한다.
업로드 시 유효성을 확인해야하므로 `handleUploadFiles` 이벤트 핸들러에 유효성 검사 로직을 추가한다.

<iframe src="https://codesandbox.io/embed/eobrodeu-pail-yuhyoseong-geomsa-zimgp?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="업로드 파일 유효성 검사"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

## 4. 프로젝트 회고

이렇게 클라이언트 상에서 `<input type="file"/>`의 파일 업로드에 대한 로딩 상태를 확인할 수 있는 Progress bar를 구성해보았다.
JS의 폼에서 직접적으로 파일을 다룰 기회가 많지 않았는데 이번 미니 프로젝트를 통해 JS로 파일을 다룰 수 있는 방법을 살펴볼 수 있는 좋은 경험이었다고 생각한다.
추가로 구성한 폼을 서버로 전송하는 로직을 구성하진 않았지만, 서버에 `formData`를 활용하여 간단하게 파일을 전송할 수 있다.

> 자세한 활용 방법은 하단 참고자료의 `formData`를 살펴보자.

<참고자료>

1. [&lt;input type="file"&gt;](https://developer.mozilla.org/ko/docs/Web/HTML/Element/Input/file)
2. [FileList](https://developer.mozilla.org/ko/docs/Web/API/FileList)
3. [FileReader](https://developer.mozilla.org/ko/docs/Web/API/FileReader)
4. [FormData](https://developer.mozilla.org/en-US/docs/Web/API/FormData)
