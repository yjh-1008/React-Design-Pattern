## Custom Hook 패턴이란?

> Custom Hook 패턴이란 특정 로직을 캡슐화하여 가독성과 유지보수성을 향상시키고, 여러 컴포넌트에서 동일한 로직을 사용할 수 있도록하는 패턴입니다.

### 언제 사용해야할까?

1. 코드의 중복 없이 컴포넌트간 로직을 공유할때
2. Props 전달이 너무 깊게 발생하여 Props Drilling이 발생할때

### 사용하지 말아야할때

1. 로직히 특정 기능에서만 사용되어 추상화가 필요하지 않을때
2. 로직이 간단하여 Hook을 생성할 필요가 없을때

### Custom Hook의 장점

1. 기능을 모듈화하기 때문에 코드의 재사용성이 높아지고 가독성이 증가합니다.
2. 로직이 캡슐화 되어있기 때문에 단위테스트를 진행할 때 더 수월하게 테스트를 진행할 수 있습니다.

## 예제

가장 일반적으로 사용되는 Axios를 사용하여 Data fetching을 하는 hook입니다.

```
import {useState, useEffect} from 'react';
import axios, {AxiosResponse, AxiosError} from 'axios';

export type ApiResponse<T> = {
  data: T | null,
  isFetching: boolean,
  error: AxiosError | null,
}

export default function useFetch<T>(url: string): ApiResponse<T> {
  const [data, setData] = useState<T|null>(null);
  const [isFetching, setIsFetching] = useState<boolean>(false);
  const [error, setError] = useState<AxiosError|null>(null);

  useEffect(() => {
    const fetch = async() => {
      try {
        setIsFetching(true);
        const res = await axios.get(url);
        setData(res.data);
      } catch(err) {
        if(axios.isAxiosError(err)) {
          setError(err);
        }
      } finally {
        setIsFetching(false);
      }
    }

    fetch();
  }, [url]);
  return {data, isFetching, error};
}

```

```
import useFetch from "../hooks/useFetch";

export default function CustomHook() {
  const {data, error, isFetching} = useFetch('api/v1/test');

  if(isFetching) {
    return <div>Loading...</div>
  }

  if(error) {
    return <div>Error: {error.message}</div>
  }

  if(!data) {
    return <div>Not Exist Data</div>
  }
  return (
    <div>
           {/* Rendering of the obtained data */}
    </div>
  )
}
```

위의 코드에서 Axios를 사용하는 로직을 분리한 다음 컴포넌트에서는 호출만 진행하고 있습니다. 각각의 상태에 따라 렌더링을 진행하고 Data Fetching을 완료한 상태에서만 정상적인 컴포넌트가 출력되게 됩니다. 위와 같이 분리를 진행하면 useAxios는 url만 변경하면 데이터 페칭이 자동으로 발생하기 때문에 응집도를 높였다고 생각합니다. 또한 이런한 방식으로 작업한다면 관심사를 분리하여 단위테스트를 더 용이하게 진행하고 코드의 명확성이 높아져서 꼭 익혀야하는 패턴이라고 생각합니다.

## 정리

프로젝트의 확장성을 위해 코드의 분리를 통해 결과적으로 관심사를 분리하는 것이 중요하다고 생각합니다. 관심사 분리를 통해 UI 기능과 데이터 처리, 로직 등의 기능을 분리하여 코드의 가독성을 높이고 Solid 원칙 중 단일 책임 원칙(SRP)를 만족시키며 로직을 구성할 수 있다고 생각합니다.
