# 2024.02.01 TIL

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

## React persist
- 코드부터 보자.
```typescript
import { configureStore } from "@reduxjs/toolkit";
import { persistStore, persistReducer } from "redux-persist";
import storage from "redux-persist/lib/storage";
import userSlice from "./userSlice";

const persistConfig = {
  key: "user",  // local storage의 user와 연결될 것
  storage,  // local storage와 연결할 것
};

// 기존 리듀서를 persistance logic로 wrap
const persistedReducer = persistReducer(persistConfig, userSlice.reducer);

const store = configureStore({
  reducer: {
    // 슬라이스-리듀서 등록 형식
    user: persistedReducer,
  },
});

// main.tsx에서 App을 감쌀 때 활용
export const persistor = persistStore(store);

// 타입 내보내기
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

export default store;

```
- 결론부터 말하자면, persist를 적용하는 데 Slice를 직접 건드릴 필요는 전혀 없다.

<적용 순서>
1. dependency install
```
npm install redux-persist
```

2. persisted state가 어떻게 handle도리 것인지 지정해주는 persistConfig를 지정한다. 난 localStorage에 'user'와 연결시켜줄 것이므로 
```typescript
const persistConfig = {
  key: "user",  // local storage의 user와 연결될 것
  storage,  // local storage와 연결할 것
};
```
위처럼 지정하였다.

3. Slice에 해당 config를 wrap 해준다.
```typescript
const persistedReducer = persistReducer(persistConfig, userSlice.reducer);

const store = configureStore({
  reducer: {
    // 슬라이스-리듀서 등록 형식
    user: persistedReducer,
  },
});
```

4. redux 상태를 rehydrate하는 process를 담당하는 persistor objefct를 지정한다.
```typescript
export const persistor = persistStore(store);
```

5. 이후, App을 해당 persistor로 감싼다.
```typescript
ReactDOM.createRoot(document.getElementById("root")!).render(
  // <React.StrictMode>
    <Provider store={store}>
      <PersistGate loading={null} persistor={persistor}>
        <App />
      </PersistGate>
    </Provider>
  /* </React.StrictMode> */
);
```