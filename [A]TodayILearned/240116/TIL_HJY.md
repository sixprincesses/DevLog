# 240116

## 스토리북

---

- 사용 목적
    - 개발자들이 컴포넌트를 독립적으로 만들고, 격리된 개발 환경에서 상호작용이 가능한 컴포넌트를 선보일 수 있도록 도와줌
    
    [[Web] Storybook를 시작해보자 (with Typescript, Next.js)](https://cheolsker.tistory.com/82)
    
- TOAST UI 컴포넌트를 스토리북에 등록
    
    ```jsx
    import type { Meta, StoryObj } from "@storybook/react";
    import InstanceMd from "./InstanceMd";
    
    const meta: Meta<typeof InstanceMd> = {
      title: "Components/Md",
      component: InstanceMd,
      tags: ["autodocs"],
      argTypes: {
        previewStyle: {
          control: "select",
          options: ["tab", "vertical"],
          description: "미리보기 선택",
        },
        initialValue: {
          control: "text",
          description: "초기값 입력",
        },
      },
    };
    
    export default meta;
    
    type Story = StoryObj<typeof InstanceMd>;
    
    export const Tab: Story = {
      args: {
        previewStyle: "tab",
        initialValue: "tab화면입니다.",
      },
    };
    
    export const Vertical: Story = {
      args: {
        previewStyle: "vertical",
        initialValue: "vertical화면입니다.",
      },
    };
    ```
    

## Custom Plugin

- 기존 마크다운 문법 이외에 문법 적용이 필요하여 custom plugin 추가
    
    ```jsx
    const gitPlugin = () => {
      const toHTMLRenderers = {
        GitPlus(node: any) {
          const body = node.literal;
    
          return [
            {
              type: "openTag",
              tagName: "div",
              outerNewLine: true,
              attributes: { style: "color:hotpink;" },
            },
            { type: "html", content: body },
            { type: "closeTag", tagName: "div", outerNewLine: true },
          ];
        },
      };
    
      return { toHTMLRenderers };
    };
    ```
    
    - node : any에서 타입 오류 해결 못함
