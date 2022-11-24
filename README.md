# EsDeKa

Communicate between `<iframe>` and host

<p align="center">
  <img src="https://github.com/pixelass/esdeka/blob/main/resources/logo.svg" alt="" width="200"/>
</p>

<!--
![Codacy coverage](https://img.shields.io/codacy/coverage/a22d58431d614c798ac08fd5414b419e?style=for-the-badge)
![Codacy grade](https://img.shields.io/codacy/grade/a22d58431d614c798ac08fd5414b419e?style=for-the-badge)
-->

## Table of Contents

<!-- toc -->

- [Purpose](#purpose)
- [Built on Zustand](#built-on-zustand)
- [Usage](#usage)
  - [Host](#host)
  - [Iframe](#iframe)
- [Typescript](#typescript)

<!-- tocstop -->

## Purpose

While developing an internal tool we wanted to communicate to third party apps/widgets that are
hosted in an iframe.

In our case we wanted to pass a state, theme and additional data down to the iframe.

## Built on Zustand

We decided to use [Zustand](https://github.com/pmndrs/zustand) as a state management library,
because it allows several mechanisms, that are required to make this possible. React does not allow
using hooks passed though an iframe to be run in the iframe, therefore we create a custom hook in
the iframe that subscribes to Zustand.

## Usage

The usage is pretty straight forward, especially if you don't use typescript.

### Host

```jsx
import { FrameWidget, SdkContextProvider, storeSlice } from "esdeka";
import create from "zustand";

const myWidget = {
  id: "my_widget",
  name: "This is my Widget ",
  widget: {
    __typename: "Frame",
    data: {
      src: "https://example.com/widgets/my-widget",
    },
  },
};

const theme = {
  primary: "#420",
};

const useStore = create(set => ({
  ...storeSlice(set),
}));

export default function App() {
  return (
    <SdkContextProvider value={{ sdk: { store: useStore, theme } }}>
      <FrameWidget sdkKey="MY_SDK" data={myWidget} />
    </SdkContextProvider>
  );
}
```

### Iframe

```jsx
import { SdkProvider, useSdk, useSdkStore } from "esdeka";

function InnerComponent() {
  // The store has a custom hook
  const widgets = useSdkStore(state => state.data);
  const setWidgetData = useSdkStore(state => state.setWidgetData);
  // The theme and data can be selected from the sdk
  const theme = useSdk(sdk => sdk.theme);
  const data = useSdk(sdk => sdk.data);
  // Deep seletors
  const id = useSdk(sdk => sdk.data.id);

  return (
    <div
      style={{
        position: "absolute",
        inset: 0,
        backgroundColor,
      }}
    >
      <button
        onClick={() => {
          setWidgetData(widget => {
            widget.message = "Hello, I am a widget";
          });
        }}
      >
        Set widget data for frame {id}
      </button>
      <pre>{JSON.stringify(widgets, null, 2)}</pre>
      <pre>{JSON.stringify(theme, null, 2)}</pre>
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}

export default function App() {
  return (
    <SdkProvider sdkKey="MY_SDK">
      <InnerComponent />
    </SdkProvider>
  );
}
```

## Typescript

We added an example in [examples/example.tsx](./examples/example.tsx). It shows how to add data to
the store and add a custom theme. Please look at the comments to understand how to add the correct
typings to your sdk. The example is a fully working sdk intended to be shared over a
package-manager. The `useStore` would usually not be shared in said package. It is intended to be
used in the host only. Only the store model (here `CounterStore`) is needed.
