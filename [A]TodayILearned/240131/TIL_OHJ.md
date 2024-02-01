# 2024.01.31 TIL

## FlipClock css -> Styled Component로 변환
- 엄청난 작업량(feat.시계 깎는 노인)
<기존의 코드 (css)>
```css
@import url('https://fonts.googleapis.com/css?family=Droid+Sans+Mono');

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

#app {
  display: flex;
  position: relative;
  width: 100%;
  min-height: 100vh;
  justify-content: center;
  align-items: center;
  background: linear-gradient(62deg, #FBAB7E 0%, #F7CE68 100%);
}

header {
  display: flex;
  position: relative;
}

h1 {
  font-family: 'Droid Sans Mono', monospace;
  font-weight: lighter;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: white;
}

.flipClock {
  display: flex;
}

.flipUnitContainer {
  width: 140px;
  height: 120px;
  perspective: 300px;
  background: white;
  border-radius: 3px;
  box-shadow: 0px 10px 10px -10px grey;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* .flipUnitContainer span {
  font-family: 'Droid Sans Mono', monospace;
  font-size: 5em;
  font-weight: lighter;
  color: lighten(black, 20%);
  backface-visibility: hidden;
} */

.upperCard,
.lowerCard {
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 100%;
  position: absolute;
  backface-visibility: hidden;
  border-radius: 5px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

span {
  font-family: 'Droid Sans Mono', monospace;
  font-size: 5em;
  font-weight: lighter;
  color: lighten(black, 20%);
  transform: translateY(0%);
  backface-visibility: hidden;
}

.flipCard {
  transform-style: preserve-3d;
  transition: transform 0.5s;
  display: flex;
  justify-content: center;
  position: absolute;
  left: 0;
  width: 100%;
  height: 50%;
  overflow: hidden;
  backface-visibility: hidden;
}

.flipCard span {
  font-family: 'Droid Sans Mono', monospace;
  font-size: 5em;
  font-weight: lighter;
  color: lighten(black, 20%);
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  backface-visibility: hidden;
}

.flipCard.unfold {
  animation: unfold 1s cubic-bezier(0.455, 0.03, 0.515, 0.955) infinite;
  transform-style: preserve-3d;
  animation-delay: 0.5s; 
  top: 50%;
  align-items: flex-start;
  transform-origin: 50% 0%;
  transform: rotateX(180deg);
  background: white;
  border-bottom-left-radius: 3px;
  border-bottom-right-radius: 3px;
  border: 0.5px solid whitesmoke;
  border-top: 0.5px solid whitesmoke;
  span {
    transform: translateY(-50%);
  }
}

.flipCard.fold {
  animation: fold 1s cubic-bezier(0.455, 0.03, 0.515, 0.955) infinite;
  transform-style: preserve-3d;
  animation-delay: 0.5s;
  top: 0%;
  align-items: flex-end;
  transform-origin: 50% 100%;
  transform: rotateX(0deg);
  background: white;
  border-top-left-radius: 3px;
  border-top-right-radius: 3px;
  border: 0.5px solid whitesmoke;
  border-bottom: 0.5px solid whitesmoke;
  span {
    transform: translateY(50%);
  }
}

@keyframes fold {
  0% {
    transform: rotateX(0deg);
  }
  100% {
    transform: rotateX(-180deg);
  }
}

@keyframes unfold {
  0% {
    transform: rotateX(180deg);
  }
  100% {
    transform: rotateX(0deg);
  }
}

.hide {
  display: none;
  transition: opacity 0.5s ease-out;
  opacity: 0;
}
```

<새 코드(tsx)>
```typescript
import styled from '@emotion/styled';
import { keyframes } from '@emotion/react';

export const AppContainer = styled.div`
  display: flex;
  position: relative;
  width: 300px;
  height: 20vh;
  justify-content: center;
  align-items: center;
`;

const foldAnimation = keyframes`
  0% {
    transform: rotateX(0deg);
  }
  100% {
    transform: rotateX(-180deg);
  }
`;

const unfoldAnimation = keyframes`
  0% {
    transform: rotateX(180deg);
  }
  100% {
    transform: rotateX(0deg);
  }
`;

export const FlipClockContainer = styled.div`
  display: flex;
  & .Fold {
    animation: ${foldAnimation} 1s cubic-bezier(0.455, 0.03, 0.515, 0.955) infinite;
    transform-style: preserve-3d;
    animation-delay: 0.5s;
    top: 0%;
    align-items: flex-end;
    transform-origin: 50% 100%;
    transform: rotateX(0deg);
    background: white;
    border-top-left-radius: 3px;
    border-top-right-radius: 3px;
    border: 0.5px solid whitesmoke;
    border-bottom: 0.5px solid whitesmoke;
  };
  & .Unfold {
    animation: ${unfoldAnimation} 1s cubic-bezier(0.455, 0.03, 0.515, 0.955) infinite;
    transform-style: preserve-3d;
    animation-delay: 0.5s;
    top: 50%;
    align-items: flex-start;
    transform-origin: 50% 0%;
    transform: rotateX(180deg);
    background: white;
    border-bottom-left-radius: 3px;
    border-bottom-right-radius: 3px;
    border: 0.5px solid whitesmoke;
    border-top: 0.5px solid whitesmoke;
  };
  & .Hide {
    display: none;
    transition: opacity 0.5s ease-out;
    opacity: 0;
  };
  & .FlipCard {
    transform-style: preserve-3d;
    transition: transform 0.5s;
    display: flex;
    justify-content: center;
    position: absolute;
    left: 0;
    width: 100%;
    height: 50%;
    overflow: hidden;
    backface-visibility: hidden;
  };
  & .FlipCard span {
    font-family: 'Droid Sans Mono', monospace;
    font-weight: lighter;
    color: lighten(black, 20%);
    display: flex;
    align-items: center;
    justify-content: center;
    height: 63.71002%;
    backface-visibility: hidden;
  }
  & .Unfold span {
    transform: translateY(-50%);
  }
  & .Fold span {
    transform: translateY(50%);
  }
  & .UpperCard {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    width: 100%;
    position: absolute;
    backface-visibility: hidden;
    border-radius: 5px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  };
  & .LowerCard {
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;
    width: 100%;
    position: absolute;
    backface-visibility: hidden;
    border-radius: 5px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  };
`;

export const FlipUnitContainer = styled.div`
  width: 80px;
  height: 60px;
  perspective: 300px;
  background: white;
  border-radius: 3px;
  box-shadow: 0px 10px 10px -10px grey;
  display: flex;
  align-items: center;
  justify-content: center;
`;

export const FlipUnitValue = styled.span`
  font-family: 'Droid Sans Mono', monospace;
  font-size: 3em;
  font-weight: lighter;
  color: lighten(black, 20%);
  backface-visibility: hidden;

`;

export const FlipCard = styled.div`

`;

export const TimerIntervene = styled.div`
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 5px 5px;
`;

```

## SSE
- 처음 axios로 SSE에 내 social Id를 보낸 후, 거기서 보내는 신호들을 받을 수 있도록 처리
    - 애초에 첫 axios도 에러가 발생할 뿐더러,
    - 이후에 SSE에서 받는 요청들도 에러가 발생함

- 다음날 (2월 1일)에 해결했는데, 핵심은
    - 애초에 SSE는 axios로 get 요청을 받는 방식이 아니다. 그냥 바로 EventSource를 통해 이벤트를 감지하고, 메시지를 받는 방식이다. 
    - 단, 여기서 문제가 발생한다. EventSource에는 headers를 설정할 수 없다는 것. 그렇다면 서버에 내 social_id와 마지막으로 메시지를 받은 시간을 어떻게 보낼 수 있을가?
    1. url 에 해당 정보를 포함해 넣는다.
    2. Event-source-polyfill이라는 라이브러리를 활용한다.
- 아쉽게도, Event-source-polyfill은 관리가 잘 되는 라이브러리가 아니라서, 타입을 지정해주는 라이브러리가 따로 없다. 따라서 타입스크립트에서 구동이 되게 하려면, 내가 따로 interface를 설정해줘야 한다.
    - 당장은 url에 해당 정보를 포함해 넣는 방식으로 성공했다.
    - 하지만 이후에 polyfill을 활용해 해결해 버릴 것!!!

```typescript
  useEffect(()=> {
    const eventSource = new EventSource(
      `http://70.12.247.83:8081/notification-service/notification/subscribe/${member_id}/${last_message}`
    );
      
    eventSource.addEventListener('message', (event) => {
      console.log('Message received:', event.data);
    });

    // Add event listener for 'error' event (optional)
    eventSource.addEventListener('error', (error) => {
      console.error('EventSource error:', error);
    });

    return () => {
      eventSource.close();
    };
  }, [member_id])
```