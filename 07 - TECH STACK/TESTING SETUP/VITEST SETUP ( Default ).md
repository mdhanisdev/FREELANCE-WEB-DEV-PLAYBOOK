
# TESTING SETUP :

# VITEST + REACT TESTING LIBRARY + MSW SETUP [ UNIT, COMPONENT & INTEGRATION TESTING ]

------------------------------------------------------------------------

## STEP 1 : Install Dependencies

```bash
pnpm add -D vitest @vitest/ui @vitest/coverage-v8 jsdom @vitejs/plugin-react
pnpm add -D @testing-library/react
pnpm add -D @testing-library/jest-dom
pnpm add -D @testing-library/user-eventś
pnpm add -D msw
```

------------------------------------------------------------------------

## STEP 2 : Create /vitest.config.ts

```ts
import { defineConfig } from "vitest/config";
import react from "@vitejs/plugin-react";
import path from "node:path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  test: {
    globals: true,
    environment: "jsdom",
    setupFiles: "./src/test/setup.ts",
    css: true,
    coverage: {
      provider: "v8",
      reporter: ["text", "html"],
      reportsDirectory: "./coverage",
      exclude: [
        "node_modules/",
        "src/test/",
        "**/*.d.ts",
        "**/*.config.*",
        ".next/",
      ],
    },
  },
});
```

------------------------------------------------------------------------

## STEP 3 : Create /src/test/handlers.ts

```ts
import { http, HttpResponse } from "msw";

export const handlers = [
  http.post("/api/login", async () => {
    return HttpResponse.json({
      success: true,
      user: {
        id: 1,
        name: "Hanis",
      },
    });
  }),
];
```

------------------------------------------------------------------------

## STEP 4 : Create /src/test/server.ts

```ts
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);
```

------------------------------------------------------------------------

## STEP 5 : Create /src/test/setup.ts

```ts
import "@testing-library/jest-dom/vitest";
import { beforeAll, afterAll, afterEach } from "vitest";
import { server } from "./server";

beforeAll(() => server.listen());

afterEach(() => server.resetHandlers());

afterAll(() => server.close());
```

------------------------------------------------------------------------

## STEP 6 : Update /tsconfig.json

```json
{
  "compilerOptions": {
    "types": ["vitest/globals"]
  }
}
```

------------------------------------------------------------------------

## STEP 7 : Update /package.json

```json
{
  "scripts": {
    "test": "vitest",
    "test:watch": "vitest --watch",
    "test:coverage": "vitest run --coverage",
    "test:ui": "vitest --ui"
  }
}
```

------------------------------------------------------------------------

## STEP 8 : Unit Test Example

`/src/utils/math.ts`

```ts
export function add(a:number,b:number){
  return a+b;
}
```

`/src/utils/math.test.ts`

```ts
import { add } from "./math";

describe("add()",()=>{
  it("adds two numbers",()=>{
    expect(add(2,3)).toBe(5);
  });
});
```

------------------------------------------------------------------------

## STEP 9 : Component Test Example

`/src/components/Button.tsx`

```tsx
type ButtonProps={children:React.ReactNode};

export function Button({children}:ButtonProps){
  return <button>{children}</button>;
}
```

`/src/components/Button.test.tsx`

```tsx
import { render,screen } from "@testing-library/react";
import { Button } from "./Button";

describe("Button",()=>{
  it("renders correctly",()=>{
    render(<Button>Login</Button>);
    expect(screen.getByRole("button")).toHaveTextContent("Login");
  });
});
```

------------------------------------------------------------------------

## STEP 10 : User Interaction Test

```tsx
import { render,screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";

function Counter(){
  const [count,setCount]=React.useState(0);
  return <>
    <p>{count}</p>
    <button onClick={()=>setCount(count+1)}>Increment</button>
  </>
}

it("increments",async()=>{
  const user=userEvent.setup();
  render(<Counter/>);
  await user.click(screen.getByRole("button"));
  expect(screen.getByText("1")).toBeInTheDocument();
});
```

------------------------------------------------------------------------

## STEP 11 : Integration Test Example

```tsx
import { render,screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { LoginForm } from "./LoginForm";

it("logs in successfully",async()=>{
  const user=userEvent.setup();

  render(<LoginForm/>);

  await user.type(screen.getByLabelText(/email/i),"hanis@test.com");
  await user.type(screen.getByLabelText(/password/i),"123456");
  await user.click(screen.getByRole("button",{name:/login/i}));

  expect(await screen.findByText(/welcome/i)).toBeInTheDocument();
});
```

> MSW intercepts the `/api/login` request and returns the mocked response from `handlers.ts`.

------------------------------------------------------------------------

## STEP 12 : Run Tests

```bash
pnpm test
```

------------------------------------------------------------------------

## STEP 13 : Watch Mode

```bash
pnpm test:watch
```

------------------------------------------------------------------------

## STEP 14 : Coverage

```bash
pnpm test:coverage
```

------------------------------------------------------------------------

## STEP 15 : View Coverage

Open:

```text
coverage/index.html
```

------------------------------------------------------------------------

## FINAL PROJECT STRUCTURE

```text
src/
├── components/
├── utils/
└── test/
    ├── handlers.ts
    ├── server.ts
    └── setup.ts

coverage/
vitest.config.ts
package.json
tsconfig.json
```
