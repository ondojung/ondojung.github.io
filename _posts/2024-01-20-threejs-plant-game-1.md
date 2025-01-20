## Greenify 1일차
화분, 흙이미지는 다 만들게 되었다. 
원래 웹용도 만들려고 했는데, 개발환경 이슈로 네이티브 앱으로 만들기로 했고 react-native를 이용해서 ts를 사용하기로 결정했다

1. 한국어로 된 자료중에 react-native자료는 부족했고 그 중에서도 reacf-native로 만든 threejs example들도 부족했기 때문에 외국 유튜브를 다 뒤져보았다.
2. 제일 처음에 겪은 문제점은 react-three/fiber/native 오류였는데 trim 어쩌구 오류가 났다. 깃허브 이슈를 뒤져서 버전을 8.16.8로 다운그레이드 했더니 해결되었다.
3. 또한 react-three/fiber/drei 에서도 orbitcontrols가 안되는 이슈가 있었는데 이 오류는 https://github.com/TiagoCavalcante/r3f-native-orbitcontrols 이 모듈을 이용해서 해결했다.

## 방 만들기
먼저 방을 만들어야 하는데, 앞쪽 벽과 바닥만 만들고 나중에 불린연산으로 창문을 뚫어서 현재 시간에 맞는 하늘과, 배경이미지를 로드할 생각이다.
또한 흙과 화분을 창문 앞에 배치할 생각이다.
먼저 '방 컴포넌트'를 제작했다. 

### 바닥부분
```js
// floor function
function Floor() {
  return (
    <mesh rotation-x={-Math.PI / 2}>
      <planeGeometry args={[20, 20]} />
      <meshBasicMaterial color={0x8b4513} />
    </mesh>
  );
}
```
코드 설명을 하자면 PlneGeometry는 일단 평면이다.
Pi/2가 90도인 것은  알 것이다.
![rotation desc](https://upload.wikimedia.org/wikipedia/commons/9/9f/Rotation_of_coordinates.svg)
처음에 벽을 세워놓으면 앞쪽으로 90도 회전한다는 뜻이다.

### 벽부분 
```js
//wall function
function Wall() {
  return (
    <mesh position={[0, 10, -10]} rotation-y={0}>
      <planeGeometry args={[20, 20]} />
      <meshStandardMaterial attach="material" color="lightblue" side={DoubleSide}/>
    </mesh>
  );
}
```
처음에 side={DoubleSide} 이부분을 추가하지 않아서 안쪽벽이 흰색이 되었는데 찾아보니 PlaneGeometry는 바깥쪽벽에 색만 기본적으로 로드한다.
그래서 side={DoubleSide} 로 설정하면 양쪽벽의 색을 둘다 로드하고, side={BackSide}는 안쪽벽을 로드한다.

전체 방 컴포넌트 코드는 다음과 같다.
```
import { DoubleSide } from "three";

function Floor() {
  return (
    <mesh rotation-x={-Math.PI / 2}>
      <planeGeometry args={[20, 20]} />
      <meshBasicMaterial color={0x8b4513} />
    </mesh>
  );
}

function Wall() {
  return (
    <mesh position={[0, 10, -10]} rotation-y={0}>
      <planeGeometry args={[20, 20]} />
      <meshStandardMaterial attach="material" color="lightblue" side={DoubleSide}/>
    </mesh>
  );
}

export default function Room(){
    return(
        <>
            <Floor />
            <Wall />
        </>
        )
}
```
![image.jpg1](https://github.com/user-attachments/assets/20f5c5e1-943a-400a-b769-c54b3f3cca6d) |![image.jpg2](https://github.com/user-attachments/assets/0fe582f8-171b-43e3-a1f0-8b5e8d8d272b) |![image.jpg3](https://github.com/user-attachments/assets/664bcf72-22fe-4c18-9414-5304e3c23837)
--- | --- | --- |

## 흙&화분컴포넌트
이 컴포넌트에서 파일을 로드할때 잘 안돼서, 꽤 고생을 했는데 require로 불러오는 확장자에 gltf파일이 없어서 그랬다.

이 문제는 expo 52에서 metro.config.js파일이 선택적으로 바뀌었는데, 터미널에서 다음 명령어로 metro.config.js파일을 생성해준다
```bash
$ npx expo customize metro.config.js
```

그리고 파일애 다음코드를 작성해서 assets파일 확장자에 .gltf, .glb확장자를 추가해서 3D파일을 불러올 수 있게한다.

```js
const { getDefaultConfig } = require("expo/metro-config");

module.exports = (() => {
  const config = getDefaultConfig(__dirname);

  const { transformer, resolver } = config;

  config.transformer = {
    ...transformer,
    babelTransformerPath: require.resolve("react-native-svg-transformer"),
  };
  config.resolver = {
    ...resolver,
    assetExts: [
      resolver.assetExts.filter((ext) => ext !== "svg"),
      "glb",
      "gltf",
      "png",
      "jpg",
      "ttf",
    ],
    sourceExts: [
      ...resolver.sourceExts,
      "svg",
      "js",
      "jsx",
      "json",
      "ts",
      "tsx",
      "cjs",
      "mjs",
    ],
  };

  return config;
})();
```
//PotAndSoil Component
```
import { useGLTF } from "@react-three/drei/native"

export default function PotAndSoil() {
  // GLTF 모델 로드
  const pot = useGLTF(require("../assets/models/pot.gltf"));
  const soil = useGLTF(require("../assets/models/dirt.gltf"));

  return (
    <>
      <primitive
        object={pot.scene}
        position={[0, 0, -5]}
        scale={[2, 2, 2]}
        rotation={[0, 0, 0]}
      />
      <primitive
        object={soil.scene}
        position={[0, 0, -5]}
        scale={[2, 2, 2]}
        rotation={[0, 0, 0]}
      />
    </>
  );
}

```
useGLTF를 활용하면 3D모델을 불러올 수 있다. 나는 흙이미지와 화분 이미지를 따로 만든 뒤, 합체를 했다.
그 후 컴포넌트를 아까만든 방 바닥에 배치를 했다.
![image.jpg1](https://github.com/user-attachments/assets/3dd9dda9-23d2-43b6-85a1-e1f5e28b250d) |![image.jpg2](https://github.com/user-attachments/assets/bfbf0fe7-739a-4799-a431-3244778f4d44)
--- | --- |

다음편에서는 Room 컴포넌트에 만든 벽에 불린연산으로 창문을 뚫고 밖에 하늘같은 맵을 만들고 날씨나 시간에 따라 시시각각 변하는 기능을 구현해볼 생각이다.