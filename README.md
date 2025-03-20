# nextjs turborepo

모노레포를 왜 사용할까? 용량이 커지지만 하나의 프로젝트를 여러 방면으로 배포해야하거나, 여러 프로젝트에서 공통적으로 사용할 수 있는 코드들을 만들어 사용하거나 디자인적으로도 공통적으로 사용하기 위해서다.

## 설치하기

```bash
npx create-turbo@latest
```

해당 명령어를 실행하면 터보레포를 통해서 모노레포 프로젝트를 시작할 수 있다.

기본적으로 **packages** 폴더와 **apps**폴더가 생성된다. 패키지 폴더에는 모노레포 내의 여러 애플리케이션에서 사용할 수 있는 공유 패키지들이 들어가 있고, apps 폴더 안에 프로젝트들이 들어가 있다.

## 패키지 만들기

필요없는 docs는 지우고, 터보레포에서 만들어주는 보일러플레이트 nextjs 앱이 항상 최신버전은 아닐 가능성이 있으니 새로 만들어보자.

```bash
cd apps
npx create-next-app@latest main-site
```

위의 명령어를 실행하면 apps 폴더로 들어가 main-site라는 nextjs 프로젝트를 생성하게 된다. 하나의 레포지토리 안에서 관리 되는 것이기 때문에 개별 프로젝트에서 불필요한 .gitignore 나 README.md 파일은 지워도 된다.

모노레포 전체에 일관성을 유지하기 위해 만든 프로젝트에 기존에 있던 구성 파일들을 이식해야한다.

### eslint 설정

```js
// apps/web/eslint.config.js
import { nextJsConfig } from "@repo/eslint-config/next-js";

/** @type {import("eslint").Linter.Config} */
export default nextJsConfig;
```

터보레포에서는 packages/eslint-config에 미리 설정을 해놓고 해당 설정을 가져오도록 되어 있다. 위의 부분을 우리가 새로 만든 main-site 프로젝트의 eslint.config.mjs에 붙여넣자.

### package.json 설정

기존에 만들어져있던 apps/web/package.json 파일을 보면 **@repo/**로 된 부분들이 있다.

```json
// apps/web/package.json

{
  "dependencies": {
    "@repo/ui": "*"
  },
  "devDependencies": {
    "@repo/eslint-config": "*",
    "@repo/typescript-config": "*"
  }
}
```

dependencies에서는 @repo/ui(packages/ui)를 사용하고, devDependencies에서는 packages/eslint-config와 packages/typescript-config를 포함하여 사용한다는 의미이다. 새로 만든 main-site에서도 이와 같이 설정해주자.

### tsconfig 설정

```json
// apps/web/tsconfig.json

{
  "extends": "@repo/typescript-config/nextjs.json",
  "compilerOptions": {
    "plugins": [
      {
        "name": "next"
      }
    ]
  },
  "include": [
    "**/*.ts",
    "**/*.tsx",
    "next-env.d.ts",
    "next.config.js",
    ".next/types/**/*.ts"
  ],
  "exclude": ["node_modules"]
}
```

기존에 만들어져 있던 apps/web/tsconfig.json 파일을 보면 packages에서 타입스크립트 설정을 갖고오는 것을 확인할 수 있다. 새로 만든 main-site 프로젝트에서도 그대로 복사해서 사용하면 된다.

### 종속성 설치

새로 만든 main-site의 설정이 끝나면 이제 종속성을 설치해줘야 한다.

```bash
# root 프로젝트로 이동 후
npm i
```

해당 명령어로 설치해주면 필수적인 패키지들이 설치되고 공유되는 패키지들도 연결될 것이다. 기본적인 설정들을 다 옮겼으면 사용하지 않을 예정인 web 프로젝트는 지워주면 된다.

```bash
npm run dev
```

dev 명령어를 실행하면 main-site 프로젝트가 실행되는 것을 확인할 수 있다.

## 공유해서 사용하기

모노레포의 장점은 코드를 다른 프로젝트들 사이에서도 공유할 수 있다는 것인데 예제를 따라해보자.

```tsx
// packages/ui/src/button.tsx

"use client";

import { ReactNode } from "react";

interface ButtonProps {
  children: ReactNode;
  className?: string;
  appName: string;
}

export const Button = ({ children, className, appName }: ButtonProps) => {
  return (
    <button
      className={className}
      onClick={() => alert(`Hello from your ${appName} app!`)}
    >
      {children}
    </button>
  );
};
```

패키지에 보면 프로젝트가 맨처음 생성되었을 때 같이 생성된 버튼이 있다.

```tsx
// apps/main-site/page.tsx

import { Button } from "@repo/ui/button";

export default function Home() {
  return (
    <main className="p-24">
      <Button
        appName="main-site"
        className="px-5 py-2 rounded-full bg-blue-800 text-white"
      >
        Click Me
      </Button>
    </main>
  );
}
```

사용할 프로젝트에서 버튼을 불러와 사용하기만 하면 된다.
