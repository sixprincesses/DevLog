# 2024.01.18 TIL

## 이미지 송신
- 유저가 jpg, jpeg, png, gif, psd, tif 가 아닌 확장자를 포함해 이미지를 송신하려고 하면 이를 막는 로직 추가
```javascript
    const allowedImageTypes = ["image/jpeg", "image/jpg", "image/png", "image/gif", "image/psd", "image/tif"];
    
    const handleInputChange = (event) => {
      event.preventDefault();
      const { name, value, type } = event.target;

      if (type === "file") {
        const file = event.target.files[0];
        
        if (file && allowedImageTypes.includes(file.type)) {
          setValues((values) => ({
            ...values,
            [name]: file,
          }));
        } else {
          setValues((values) => ({
            ...values,
            [name]: null,
          }));
          console.log("Invalid file type. Only jpg, jpeg, png, gif, psd, and tif are allowed.");
        }
      } else {
        setValues((values) => ({
          ...values,
          [name]: value,
        }));
      }
    };
```


## 이미지 수신
- 프론트에서는 비교적 간단한 로직 (fetch 활용)

```javascript
    useEffect(() => {
      const fetchData = async () => {
        try {
          const response = await fetch("http://70.12.247.83:8080/member/2", {
            method: "GET",
          });
  
          if (response.ok) {
            const data = await response.json();
            console.log(data);
          } else {
            const data = await response.json();
            console.log(data);
            console.error("Failed to fetch data");
          }
        } catch (error) {
          console.error("Error occurred:", error);
        }
      };
      fetchData();
    }, []); 
```


- 에러가 발생했을 시 왜 에러 메시지가 전송이 안 되는가?
    - 내가 에러 발생 시 response body를 수신하도록 설정을 안 해놨다
