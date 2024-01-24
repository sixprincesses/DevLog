# 2024.01.24 TIL

## Redux Toolkit 적용

```javascript
interface userState {
  social_id: number;
  social_nickname: string;
  name: string;
  phone: string;
  email: string;
  gender: string;
  image_id: string;
  access_token: string;
}

const initialState: userState = {
  social_id: 0,
  social_nickname: "ktg",
  name: "김태규",
  phone: "010-9753-1677",
  email: "xorb269@naver.com",
  gender: "남성",
  image_id:
    "https://png.pngtree.com/png-vector/20191115/ourmid/pngtree-beautiful-profile-line-vector-icon-png-image_1990469.jpg",
  access_token: "123"
};

const userSlice = createSlice({
  // 슬라이스의 이름
  name: "userSlice",
  // 상태 초기값
  initialState,
  // 리듀서(상태 관리 매서드)
  reducers: {
    login: (state, action: PayloadAction<userState>) => {
      // action.payload에 로그인 된 사용자 정보가 들어올 예정
      return {
        ...state,
        ...action.payload,
      };
    },
  },
});
```
- userSlice.tsx 에서 유저 정보와 access_token 등을 받고 관리할 것

```javascript
const token = useAppSelector((state) => state.user.access_token )
  const dispatch = useAppDispatch();

  const getAccessToken = async (authorizationCode: string) => {
    try {
      const callbackURL: string = 'http://70.12.247.83:8080/member-service/member/login';
      const response = await axios.post(callbackURL, {
        code: authorizationCode
      }, {
        headers: {
          'Content-Type': 'application/json',
        }
      });
  
      const newAccessToken: string = response.data.access_token;
      
      // Include access token in the payload when dispatching the login action
      dispatch(login({
        ...response.data,
        access_token: newAccessToken,
      }));
  
      setIsLogin(true);
      setAccessToken(newAccessToken);
      console.log(response, newAccessToken);
    } catch (error) {
      console.error('Error getting access token:', error);
    }
  };
```
- 로그인 페이지에서 dispatch를 통해 중앙 store에 저장되어있는 access_token을 업데이트해주는 로직.
- 업데이트 되는 것을 확인

### UseSelector에 대한 고찰
- 팀원들이 UseSelector를 활용할 때 구조분해 할당을 통해 상태를 가져오는 것을 선호한다고 함
    - 이 경우, 상태 하나가 변경되었을 시 전체적으로 useSelector로 구조분해 할당되어 있는 모든 상태들에 대해 리렌더링이 일어나기 때문에 성능이 저하될 여지가 있음
    - 이 부분은 추후 지적할 것