# 240110

# ToastUI

## 사용 목적

---

- 공부 페이지에 마크다운을 넣어 주기 위한 마크다운 에디터가 필요

## 사용 방법

---

- 설치 및 import
    - 설치
        
        ```jsx
        npm i --save @toast-ui/react-editor
        ```
        
    - 발생한 오류
        
        ```jsx
        npm ERR! code ERESOLVE
        npm ERR! ERESOLVE could not resolve
        npm ERR! 
        npm ERR! While resolving: @toast-ui/react-editor@3.2.3
        npm ERR! Found: react@18.2.0
        npm ERR! node_modules/react
        npm ERR!   react@"^18.2.0" from the root project
        npm ERR!   peer react@"^18.2.0" from react-dom@18.2.0
        npm ERR!   node_modules/react-dom
        npm ERR!     react-dom@"^18.2.0" from the root project
        npm ERR! 
        npm ERR! Could not resolve dependency:
        npm ERR! peer react@"^17.0.1" from @toast-ui/react-editor@3.2.3
        npm ERR! node_modules/@toast-ui/react-editor
        npm ERR!   @toast-ui/react-editor@"^3.2.3" from the root project
        npm ERR! 
        npm ERR! Conflicting peer dependency: react@17.0.2
        npm ERR! node_modules/react
        npm ERR!   peer react@"^17.0.1" from @toast-ui/react-editor@3.2.3
        npm ERR!   node_modules/@toast-ui/react-editor
        npm ERR!     @toast-ui/react-editor@"^3.2.3" from the root project
        npm ERR! 
        npm ERR! Fix the upstream dependency conflict, or retry
        npm ERR! this command with --force or --legacy-peer-deps
        npm ERR! to accept an incorrect (and potentially broken) dependency resolution.
        npm ERR! 
        npm ERR! 
        npm ERR! For a full report see:
        npm ERR! /Users/juyoung/.npm/_logs/2024-01-09T23_43_10_689Z-eresolve-report.txt
        ```
        
        - node 버전 문제 → --force로 해결
    - import
        
        ```jsx
        import '@toast-ui/editor/dist/toastui-editor.css';
        import { Editor } from '@toast-ui/react-editor';
        ```
        
- 구현 코드
    
    ```jsx
    import "@toast-ui/editor/dist/toastui-editor.css";
    import "@toast-ui/editor/dist/theme/toastui-editor-dark.css";
    import { useRef } from "react";
    
    import { Editor } from "@toast-ui/react-editor";
    
    const MdEditor = () => {
      const editorRef = useRef(null);
    
      const handleSave = () => {
        const markDownContent = editorRef.current.getInstance().getMarkdown();
        if (markDownContent.length < 10) {
          editorRef.current.getInstance().focus(); // editor에 focus
          return;
        }
        console.log(markDownContent);
      };
    
      return (
        <div className="MarkDownEditor" style={{ width: "80vw", margin: "80px" }}>
          <h2>Toast UI Test</h2>
          <Editor
            ref={editorRef}
            placeholder="여기에 입력 해주세요."
            previewStyle="vertical"
            initialEditType="markdown"
            hideModeSwitch={true}
            theme="dark"
            usageStatistics={false}
            initialValue="# 여기에 제목을 입력해주세요"
          />
          <button
            className="MarkDownSendingBtn"
            style={{
              margin: "20px",
              backgroundColor: "greenyellow",
              color: "green",
            }}
            onClick={handleSave}
          >
            저장하기
          </button>
        </div>
      );
    };
    
    export default MdEditor;
    ```
    
    - 기능
        - 마크다운 문장 스트링으로 콘솔 찍어보기
        - 10기자 미만하고 요청시 다시 에디터에 포커스

## 참고자료

---

- markdown editor 제공해주는 라이브러리

[Editor](https://ui.toast.com/tui-editor)

- 기본사용법

[[React] 마크다운 에디터 사용하기 Toast UI Editor / useRef, forwardRef](https://velog.io/@y0ungg/React-마크다운-에디터-사용하기-Toast-UI-Editor)

- 콘텐츠 가져오기 구현

[리액트 - Toast UI 에디터 저장하고 초기화하기 (Save and Initialize Toast UI Editor)](https://bloodstrawberry.tistory.com/1271)

- Editor에 focus하기

[리액트 toast ul editor(TypeError: textRef.current.focus is not a function. (In 'textRef.current.focus()', 'textRef.current.focus' is undefined) 에러 해결](https://velog.io/@h-young/리액트-toast-ul-editorTypeError-textRef.current.focus-is-not-a-function.-In-textRef.current.focus-textRef.current.focus-is-undefined-에러-해결)

- 사진 첨부 해결

[[React] TOAST UI Editor 사진 첨부 문제](https://velog.io/@matajeu/React-TOAST-UI-Editor-사진-첨부-문제)

## 해결 못한 문제

---

- 마크다운 to pdf 필요
- 토스트 UI에 이미지 어떻게?
    - 드래그앤 드랍?
    - 현재 방식 서버 터짐
