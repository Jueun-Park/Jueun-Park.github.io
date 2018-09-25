---
layout: post
title: C언어 문제풀이를 위한 VS Code 설정 기록
published: True
---



Window10 환경에서 VS Code를 사용해보려고 한다. 백준 등 온라인 저지 문제를 풀기 위해 다음과 같이 세팅하고 싶었다.

1. C 사용
2. gcc 컴파일러 사용
3. 작업 중인 디렉토리 안에 `bin` 디렉토리를 만들어 빌드된 `.exe` 파일 보관
4. `ctrl+shift+b`로 빌드
5. `ctrl+f5`로 디버깅 없이 실행
6. `f5`로 gdb를 이용해 디버깅

이 글은 미래의 내가 설정 방법을 까먹을 것을 대비해서 작성했다. ~~그런데 쓰다 보니 다른 분들에게도 써 보라고 권유하는 것처럼 되었다~~



# VS Code 설치

VS Code는 [Electron](https://electronjs.org/)이라는 신기한 기술로 만들어진 IDE이다. 다양한 확장 프로그램으로 여러 가지 언어를 지원한다. 가볍고 이쁘다. 내가 원하는 기능만 설치하고 사용하지 않는 기능은 쉽게 삭제할 수 있다. `보기 - Zen모드`는 화면에 오직 코드만을 띄워 준다. 그 외에도 세로로 긴 직사각형 블럭 지정이나 커서 복제 기능은 아직 안 써봤지만 처음 보고는 유용할 것이라는 생각을 했다. [vscode를 써봐야겠다고 마음 먹게 해 준 글](https://www.vobour.com/%EA%B0%9C%EB%B0%9C-%EC%83%9D%EC%82%B0%EC%84%B1%EC%9D%84-%EC%98%AC%EB%A0%A4%EC%A3%BC%EB%8A%94-vscode%EC%9D%98-%EC%86%8C%EC%86%8C-%ED%95%9C-%EA%B8%B0%EB%8A%A5%EB%93%A4)

[Visual Studio Code](https://code.visualstudio.com/)에서 다운로드하고 설치한다. 처음 실행하면 **한글 언어팩 확장 프로그램**을 설치하겠냐고 묻는다. 친절하다. 그것과 디버깅을 쉽게 해 줄 **C/C++ 확장 프로그램**을 설치한 뒤 `다시 로드`버튼을 눌러 활성화시킨다.



# MinGW 설치

gcc를 사용하기 위해 설치한다. [MinGW](http://www.mingw.org/) 홈페이지에서 다운로드한다. 설치 후 환경 변수에 등록한 뒤 재부팅한다.



# 작업 디렉토리 만들기

나는 `problem-solving-practices` 디렉토리를 만들었다. VS Code를 실행한 뒤 `파일 - 폴더 열기`를 통해 해당 폴더를 연다. 그 안에 예제로 `1000.c` 파일을 만들었다. 다음이 [boj 1000번](https://www.acmicpc.net/problem/1000) 예제 코드이다.

```c
#include <stdio.h>

int main() {
    int a, b;
    scanf("%d %d", &a, &b);
    printf("%d\n", a + b);
    return 0;
}
```



# 빌드 작업 구성

다 작성했으면 빌드해야 한다. 작성한 코드 창에서 `ctrl+shift+b`를 누른다. 바로 빌드될 것이라는 생각과는 달리 뭐가 없다고 한다. 새로 뜬 `빌드 작업 구성` 버튼을 눌러본다. `템플릿에서 tasks.json 만들기` - `Others` 를 선택해 `./.vscode/tasks.json`을 만든다.

## `tasks.json` 

나는 여러 가지 삽질을 통해 다음과 같은 설정 코드를 쓰기로 했다. [이 블로그의 글](http://webnautes.tistory.com/1158)과 [공식 문서](https://code.visualstudio.com/Docs/editor/tasks#_typescript-hello-world)를 가장 많이 참고했다.

```json
{
    "version": "2.0.0",
    "runner": "terminal",
    "type": "shell",
    "echoCommand": true,
    "presentation" : { "reveal": "always" },    
    "tasks": [
        // build task
        {
            "label": "build c",
            "command": "gcc",
            "args": [
                "${file}",
                "-g",  // debug option, 디버깅 안 할 거라면 없애도 된다(실행파일 용량이 작아짐)
                "-o",
                // 해당 디렉토리 내에 bin 디렉토리를 미리 생성해주어야 한다
                "${fileDirname}\\bin\\${fileBasenameNoExtension}"
            ],
            "group": {
                "kind": "build",
                "isDefault": true,
                },
            
            //컴파일시 에러를 편집기에 반영 (problem matcher)
            "problemMatcher": {
                "fileLocation": [
                    "relative",
                    "${workspaceRoot}"
                ],
                "pattern": {
                    "regexp": "^(.*):(\\d+):(\\d+):\\s+(warning error):\\s+(.*)$",
                    "file": 1,
                    "line": 2,
                    "column": 3,
                    "severity": 4,
                    "message": 5
                }
            },
        },

        // test task
        {
            "label": "execute",
            "command": "powershell",
            "args": [
                "/C",
                "${fileDirname}\\bin\\${fileBasenameNoExtension}"
            ],
            "group": {
                "kind": "test",
                "isDefault": true,
                },
        },
    ]
}
```

내가 원하는 *단축키 누르자마자 작업 실행*을 위해 `"isDefault": true` 부분을 추가했다.

# 단축키 덮어쓰기

이제 `ctrl+shift+b`를 누르면 빌드 작업이 실행된다. 위 코드에서 `"group"`의 이름이 `"build"`인 부분만이 실행되는 것이다. 그럼 `"test"` 작업은 어떻게 실행하느냐? 내가 원하는 단축키를 골라 키 바인딩을 수정해야 한다. 나는 `ctrl+f5`를 쓰고 싶었다. `파일 - 기본 설정 - 바로 가기 키` 를 들어간다. 그럼 `keybindings.json` 파일을 수정할 수 있다는 안내가 나오고 누르면 해당 파일이 열린다. 다음과 같이 수정한다.

```json
// Place your key bindings in this file to overwrite the defaults
[
    {
        "key": "ctrl+f5",
        "command": "workbench.action.tasks.test"
    }
]
```

사실 원래 vscode의 Default Keybindings에는 이렇게 되어 있다. 이것을 덮어 쓰는 것이다... ~~왜냐하면 나는 이 커맨드에 대응하는 작업을 설정하는 법을 못 찾았기 때문이다~~

```json
{ "key": "ctrl+f5",
  "command": "workbench.action.debug.run",
  "when": "!inDebugMode" },
```



## 코드 실행

이제 `ctrl+shift+b`를 눌러 빌드하고, `ctrl+f5`를 누르면 빌드된 프로그램이 실행될 것이다.



# 디버깅

디버깅 모드가 실행되는지 보기 위해 예제 코드의 `return 0;`에 중단점~~빨간콩~~을 찍고 `f5`를 눌러 본다. 그러면 역시 뭐가 없는지 환경을 설정하라고 한다. `C++ (GDB/LLDB)`를 선택한다. 새 json 파일이 생성되었다.

참고로 `f5` 키는 다음과 같이 바인딩 되어 있다.

```json
{ "key": "f5",
  "command": "workbench.action.debug.start",
  "when": "!inDebugMode" },
```



## `launch.json`

새로 생긴 `launch.json` 파일을 보자. 주어진 템플릿의,

```
"program": "enter program name, for example ${workspaceFolder}/a.exe",
...
"miDebuggerPath": "/path/to/gdb",
```

이 두 부분을 수정해주면 되는 것 같다. 나는 다음과 같이 변경했다.

```json
"program": "${workspaceFolder}/bin/${fileBasenameNoExtension}.exe",
...
"miDebuggerPath": "C:/MinGW/bin/gdb.exe",
```

이제 `f5` 를 누르면 디버깅이 시작된다. `f5`를 누를 때마다 다음 중단점으로 넘어간다.

## 주의할 점

gcc 컴파일 시 `-g` 옵션을 넣어 주어야 디버깅 시 필요한 정보를 포함하여 빌드된다. 위의 `tasks.json` 코드에서 그 부분을 확인할 수 있다.



# 그 외

#### 테마

`파일 - 기본 설정 - 색 테마` 에서 색 테마를 바꿀 수 있다. 개인적으로 monokai가 아름답다고 생각한다. 글꼴은 기본 설정도 한글이 괜찮게 나와서 특별히 바꾸지 않았다...

#### 확장 프로그램

확장 프로그램이 아주 많다고 한다. 사용하다가 '이런 기능 없나?'하는 생각이 들 때마다 받아서 사용해 볼 생각이다. 마치 뷔페 같다. ~~당신이 뷔페를 좋아한다면 vscode도 좋아하게 될 것이다~~

#### 서브라임 텍스트와의 관계?

가볍고 확장 프로그램으로 기능을 추가하며 사용한다는 점이 비슷하다는데, 둘 다 깊게 사용해보지 않아서 차이가 어떤지 알지 못한다. (누가 알려주셨으면...) 내가 느끼기에 서브라임 텍스트3은 예전에 처음 깔았을 때 디렉토리 탐색기가 붙어 있지 않아서 검정 메모장(...) 같다고 생각했었다. (추가: [이 글](https://demun.github.io/vscode-tutorial/)에 따르면 vscode는 "특히 서브라임텍스트의 한글입력 문제, 인코딩 문제를 깔끔히 해결한 에디터"라고 한다.)

#### 다른 언어

Python 개발 환경도 vscode로 구성할 수 있다고 하는데, 묵직한 Pycharm이 조금 더 마음에 안 드는 날이 오면 한 번 시도해볼지도 모르겠다.