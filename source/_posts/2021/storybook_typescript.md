---
title: " [實作筆記] React 與 Storybook 開發"
date: 2021/01/18 16:01:35
tags:
  - TypeScript
  - React
---

## 前情提要

不論是 App 或是 Web, 與使用者第一線互動的就是 UI 了。  
另一面在需求設計上, 我們總會想像一個畫面，  
想像著使用者如何使用我們的產品，  
也就是說 UI 是理想與真實的邊界。

Designer 完成了設計, Engineer 將之實作出來,  
主流的開發方式會透過 Component 來節省時間。

## 為什麼我們需要 Storybook ?

但是真的能節省時間嗎 ?

開發人員彼此之間會不會重複造輪子? 他們又要怎麼溝通?  
修改到底層元件會不會影響到上層元件? 會不會改 A 壞 B?  
複雜的 Component, 特殊的情境如何測試 ?

Storybook 恰恰能解決這些問題,

- 作為開發人員的指南和文件
- 獨立於應用程式建立 Component
- 測試特殊情境

對我來說，最重要的事，我可以用類似 TDD 的方式開發,  
在 Storybook 的官方文件提到這個方法為 CDD.
在 TDD 中我們把一個個 Use Case 寫成 Test Case,  
我們可以挪用這個觀念,  
在 Storybook 中把每一個 Component 的各種狀態(State),  
當作 Use Case, 然後透過 Mock State 讓 Component 呈現該有的樣貌。

## 心得

大前端的時代，僅僅只看 Web 的話，
我認為這個時代前端的重心就在兩個主要的技術之上，
Component 與 State Management。
而實作你可以有以下的選擇,  
僅介紹我聽過的主流 Library,  
Component 與 State Management 沒有絕對的搭配關係。

| Component                      | State Management                           |
| ------------------------------ | ------------------------------------------ |
| [React](https://reactjs.org/)  | [Flux](https://facebook.github.io/flux/)   |
| [Angular](https://angular.io/) | [Redux](https://redux.js.org/)             |
| [Vue](https://vuejs.org/)      | [Akita](https://datorama.github.io/akita/) |

## 改編 Storybook 教程

為什麼要改編 [Storybook 教程(React Version)](https://www.learnstorybook.com/intro-to-storybook/react/en/get-started/) ?

這個教程會以一個簡單的 Todo List,  
從創建應用程式、簡單的 Component 到複雜,  
與狀態管理器介接, 測試到部署。

但是他缺了一味，[TypeScript](https://www.typescriptlang.org/),  
所以我自已用 TypeScript 進行了改寫並稍作一下記錄。

### 環境

- 作業系統 : [Windows 10 Pro](https://www.microsoft.com/zh-tw/windows/compare-windows-10-home-vs-pro)
- 瀏覽器 : Chrome

### 開始

設定初始化的環境

#### 設定 React Storybook

開啟命令提示視窗，執行以下命令以創建 React App

```cmd
# Create our application:
npx create-react-app taskbox

cd taskbox
```

安裝 Storybook

```cmd
npm i storybook

# Add Storybook:
npx -p @storybook/cli sb init
```

啟動開發環境的 Storybook，

```cmd
# Start the component explorer on port 6006:
yarn storybook
```

測試與執行

```cmd
# Run the test runner (Jest) in a terminal:
yarn test --watchAll

# Run the frontend app proper on port 3000:
yarn start
```

![npm Storybook](https://i.imgur.com/uQXmVvQ.jpg)

下載 [CSS](https://github.com/chromaui/learnstorybook-code/edit/master/src/index.css),存檔至 src/index.css

安裝 degit

```cmd
npm i degit
```

加入 Add assets (字型與 Icon)

```cmd
npx degit chromaui/learnstorybook-code/src/assets/font src/assets/font
npx degit chromaui/learnstorybook-code/src/assets/icon src/assets/icon
```

Git Commit

```cmd
> git add .
> git commit -m "first commit"
```

#### 簡單的 component

在 `src/components/` 資料夾建立 component `Task.js`

```javascript=
// src/components/Task.js

import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className="list-item">
      <input type="text" value={title} readOnly={true} />
    </div>
  );
}
```

建立 Task.stories.js

```javascript=
// src/components/Task.stories.js

import React from 'react';
import Task from './Task';

export default {
  component: Task,
  title: 'Task',
};

const Template = args => <Task {...args} />;

export const Default = Template.bind({});
Default.args = {
  task: {
    id: '1',
    title: 'Test Task',
    state: 'TASK_INBOX',
    updatedAt: new Date(2018, 0, 1, 9, 0),
  },
};

export const Pinned = Template.bind({});
Pinned.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_PINNED',
  },
};

export const Archived = Template.bind({});
Archived.args = {
  task: {
    ...Default.args.task,
    state: 'TASK_ARCHIVED',
  },
};
```

隨時你都可以執行 `yarn storybook` 試跑來看看 storybook
調整 Storybook 的 config 檔 (.storybook/main.js)

```javascript=
// .storybook/main.js

module.exports = {
  stories: ['../src/components/**/*.stories.js'],
  addons: [
    '@storybook/addon-links',
    '@storybook/addon-essentials',
    '@storybook/preset-create-react-app',
  ],
};
```

(.storybook/preview.js) 這設定為了 log UI 上的某些操作產生的事件，
在之後我們會看到 **完成(`onArchiveTask`)或置頂(`onPinTask`)** 兩個事件

```javascript=
// .storybook/preview.js

import '../src/index.css';

// Configures Storybook to log the actions(onArchiveTask and onPinTask) in the UI.
export const parameters = {
  actions: { argTypesRegex: '^on[A-Z].*' },
};
```

調整 `Task.js`

```javascript=
// src/components/Task.js

import React from 'react';

export default function Task({ task: { id, title, state }, onArchiveTask, onPinTask }) {
  return (
    <div className={`list-item ${state}`}>
      <label className="checkbox">
        <input
          type="checkbox"
          defaultChecked={state === 'TASK_ARCHIVED'}
          disabled={true}
          name="checked"
        />
        <span className="checkbox-custom" onClick={() => onArchiveTask(id)} />
      </label>
      <div className="title">
        <input type="text" value={title} readOnly={true} placeholder="Input title" />
      </div>

      <div className="actions" onClick={event => event.stopPropagation()}>
        {state !== 'TASK_ARCHIVED' && (
          // eslint-disable-next-line jsx-a11y/anchor-is-valid
          <a onClick={() => onPinTask(id)}>
            <span className={`icon-star`} />
          </a>
        )}
      </div>
    </div>
  );
}
```

加入測試用的外掛(add on)

```cmd
yarn add -D @storybook/addon-storyshots react-test-renderer
```

執行測試

```cmd
> yarn test
```

測試結果

```terminal
yarn run v1.22.0
$ react-scripts test
(node:52888) DeprecationWarning: \`storyFn\` is deprecated and will be removed in Storybook 7.0.

https://github.com/storybookjs/storybook/blob/next/MIGRATION.md#deprecated-storyfn
 PASS  src/components/storybook.test.js (14.703 s)
  Storyshots
    Task
      √ Default (13 ms)
      √ Pinned (2 ms)
      √ Archived (1 ms)

 › 3 snapshots written.
Snapshot Summary
 › 3 snapshots written from 1 test suite.

Test Suites: 1 passed, 1 total
Tests:       3 passed, 3 total
Snapshots:   3 written, 3 total
Time:        16.716 s
Ran all test suites related to changed files.
```

#### 簡單的 component 改用 typescript

首先，`Task.js` 調整副檔名為 `Task.tsx`，  
`Task.stories.js` 為 `Task.stories.tsx`.  
測試檔案 `storybook.test.js` 也一併修改 `storybook.test.ts`

並修改 `.storybook/main.js`

```javascript=
module.exports = {
  stories: ['../src/components/**/*.stories.tsx'],
  /// 略…
};
```

建立 `tsconfig.json` 檔

```cmd
> tsc --init
```

用 TypeScript 改寫

```typescript=
// src/components/Task.tsx
import React from 'react';

export enum TaskState{
  Inbox = 'TASK_INBOX',
  Pinned = 'TASK_PINNED',
  Archived = 'TASK_ARCHIVED'
}

export interface TaskArgs {
  item:TaskItem,
  onArchiveTask: (id:string) => void,
  onPinTask: (id:string) => void
}

export class TaskItem{
  id: string = ''
  title: string = ''
  state: TaskState = TaskState.Inbox
  updatedAt?: Date
}

export default function Task(args:TaskArgs) {
  return (
    <div className={`list-item ${args.item.state}`}>
      <label className="checkbox">
        <input
          type="checkbox"
          defaultChecked={args.item.state === TaskState.Archived}
          disabled={true}
          name="checked"
        />
        <span className="checkbox-custom" onClick={()=>args.onArchiveTask(args.item.id)} />
      </label>
      <div className="title">
        <input type="text" value={args.item.title} readOnly={true} placeholder="Input title" />
      </div>

      <div className="actions" onClick={event => event.stopPropagation()}>
        {args.item.state !== TaskState.Archived && (
          // eslint-disable-next-line jsx-a11y/anchor-is-valid
          <a onClick={()=>args.onPinTask(args.item.id)}>
            <span className={`icon-star`} />
          </a>
        )}
      </div>
    </div>
  );
}
```

改寫 Task.store.tsx

```typescript=
// src/components/Task.stories.tsx

import React from 'react';
import Task, { TaskItem, TaskArgs, TaskState } from './Task';
import { Story } from '@storybook/react/types-6-0';

export default {
  component: Task,
  title: 'Task',
};

const Template:Story<TaskArgs> = args => <Task {...args} />;

var defaultItem:TaskItem = {
  id:'1',
  title:'Test Task',
  state:TaskState.Inbox,
  updatedAt: new Date(2018, 0, 1, 9, 0),
};

export const Default = Template.bind({});
Default.args = { item: defaultItem, };

export const Pinned = Template.bind({});
var pinnedItem = Copy(defaultItem);
pinnedItem.state=TaskState.Pinned
Pinned.args = { item: pinnedItem };

export const Archived = Template.bind({});
var archivedItem = Copy(defaultItem);
archivedItem.state=TaskState.Archived;
Archived.args = {item: archivedItem};

function Copy(obj:any) {
  return Object.assign({},obj);
}
```

#### 組合成複雜的 component (TypeScript 版本)

與教程最主要的不同之處在於使用了 TypeScript 的語法撰寫

```typescript==
 // src/components/TaskList.tsx
import React from 'react';
import Task, { TaskItem, TaskState } from './Task';
import { connect } from 'react-redux';
//import { archiveTask, pinTask } from '../lib/redux';

export interface TaskListProps {
  loading?:boolean;
  tasks: TaskItem[];
  onArchiveTask: (id:string)=>void;
  onPinTask: (id:string)=>void;
}

export function PureTaskList(props:TaskListProps) {
  const events = {
    onArchiveTask:props.onArchiveTask,
    onPinTask:props.onPinTask,
   };

  const LoadingRow = (
    <div className="loading-item">
      <span className="glow-checkbox" />
      <span className="glow-text">
        <span>Loading</span> <span>cool</span> <span>state</span>
      </span>
    </div>
  );

  if (props.loading) {
    return (
      <div className="list-items">
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
        {LoadingRow}
      </div>
    );
  }

  if (props.tasks === undefined || props.tasks.length === 0) {
    return (
        <div className="list-items">
            <div className="wrapper-message">
                <span className="icon-check" />
                <div className="title-message">You have no tasks</div>
                <div className="subtitle-message">Sit back and relax</div>
            </div>
        </div>
    );
  }

  const tasksInOrder = [
    ...props.tasks.filter(t => t.state === TaskState.Pinned), //< ==== 固定頂部
    ...props.tasks.filter(t => t.state !== TaskState.Pinned),
  ];


  return (
    <div className="list-items">
      {tasksInOrder.map(item => (
        <Task key={item.id} item={item} {...events}/>
      ))}
    </div>
  );
}

export default connect(
  (props:TaskListProps) => ({
    tasks: props.tasks.filter(t => t.state === TaskState.Inbox || t.state === TaskState.Pinned ),
  })
)(PureTaskList);
```

`TaskList.stories.tsx` 設置，也是使用 TypeScript 撰寫。

```typescript=
// src/components/TaskList.stories.tsx

import React from 'react';
import { TaskList, TaskListArgs } from './TaskList';
import { TaskItem, TaskState } from './Task';
import { Story } from '@storybook/react/types-6-0';

export default {
  component: TaskList,
  title: 'TaskList',
  decorators: [(story: () => React.ReactNode) => <div style={{ padding: '3rem' }}>{story()}</div>],
  excludeStories: /.*Data$/,
};

const Template:Story<TaskListArgs> = args => <TaskList {...args} />

var defaultItem:TaskItem = {
  id:'1',
  title:'Test Task',
  state:TaskState.Inbox,
  updatedAt: new Date(2018, 0, 1, 9, 0)
};

export const Default = Template.bind({});
Default.args = {
  tasks: [
    { ...defaultItem, id: '1', title: 'Task 1' },
    { ...defaultItem, id: '2', title: 'Task 2' },
    { ...defaultItem, id: '3', title: 'Task 3' },
    { ...defaultItem, id: '4', title: 'Task 4' },
    { ...defaultItem, id: '5', title: 'Task 5' },
    { ...defaultItem, id: '6', title: 'Task 6' },
  ],
};

export const WithPinnedTasks = Template.bind({});
WithPinnedTasks.args = {
  tasks: [
    ...Default.args.tasks!.slice(0,5),
    { id: '6', title: 'Task 6 (pinned)', state: TaskState.Pinned },
  ],
};

export const Loading = Template.bind({});
Loading.args = {
  tasks: [],
  loading: true,
};

export const Empty = Template.bind({});
Empty.args = {
  ...Loading.args,
  loading: false,
};
```

#### 介接 Store 資料

建立 Redux

```typescript=
// src/lib/redux.ts

// A simple redux store/actions/reducer implementation.
// A true app would be more complex and separated into different files.
import { createStore } from 'redux';
import { TaskItem, TaskState } from '../components/Task';

export const archiveTask = (id: string) => {
  console.log("archive task:"+id);
  return ({ type: TaskState.Archived, id })
};

export const pinTask = (id: string) => {
  console.log("pin task:"+id);
  return ({ type: TaskState.Pinned, id })
};

// The reducer describes how the contents of the store change for each action
export const reducer = (state: any, action: { id:string; type: TaskState; }) => {
  switch (action.type) {
    case TaskState.Archived:
    case TaskState.Pinned:
      return taskStateReducer(action.type)(state, action);
    default:
      return state;
  }
};

// The initial state of our store when the app loads.
// Usually you would fetch this from a server
const defaultTasks:Array<TaskItem> = [
  { id: '1', title: 'Something', state: TaskState.Inbox },
  { id: '2', title: 'Something more', state: TaskState.Inbox },
  { id: '3', title: 'Something else', state: TaskState.Inbox },
  { id: '4', title: 'Something again', state: TaskState.Inbox },
];


// We export the constructed redux store
export default createStore(reducer, { tasks: defaultTasks });

// All our reducers simply change the state of a single task.
function taskStateReducer(taskState: TaskState) {
  return (state: { tasks: TaskItem[]; }, action: { id: string; }) => {
    return {
      ...state,
      tasks: state.tasks.map(task =>
        task.id === action.id ? { ...task, state: taskState } : task
      ),
    };
  };
}
```

修改 TaskList.tsx 視作一個 container 與 redux 作介接:

```typescript=
// src/components/TaskList.tsx
import React from 'react';
import Task, { TaskItem, TaskState } from './Task';
import { connect } from 'react-redux';
import { archiveTask, pinTask } from '../lib/redux';

// 中略...

export default connect(
  (props:TaskListArgs) => ({
    tasks: props.tasks.filter(t => t.state === TaskState.Inbox || t.state === TaskState.Pinned ),
  }),
  dispatch => ({
    onArchiveTask: (id: string) => dispatch(archiveTask(id)),
    onPinTask: (id: string) => dispatch(pinTask(id)),
}))(TaskList);
```

加上 Page `InboxScreen`

```typescript=
//src/components/InboxScreen.js

import React from 'react';
import { connect } from 'react-redux';
import TaskList from './TaskList';

export class InboxScreenArgs {
  error:string | undefined
}

export function InboxScreen(args:InboxScreenArgs) {
  if (args.error) {
    return (
      <div className="page lists-show">
        <div className="wrapper-message">
          <span className="icon-face-sad" />
          <div className="title-message">Oh no!</div>
          <div className="subtitle-message">Something went wrong</div>
        </div>
      </div>
    );
  }

  return (
    <div className="page lists-show">
      <nav>
        <h1 className="title-page">
          <span className="title-wrapper">TaskBox</span>
        </h1>
      </nav>
      <TaskList />
    </div>
  );
}

export default connect((props:InboxScreenArgs) => (props))(PureInboxScreen);
```

一樣也加上 Story ,`InboxScreen.stories.tsx`  
讓我們可以透過 Storybook 作人工 E2E 測試

```typescript=
//src/components/InboxScreen.stories.tsx

import React from 'react';
import { Provider } from 'react-redux';
import { InboxScreenArgs, InboxScreen } from './InboxScreen';
import { Story } from '@storybook/react/types-6-0';
import store from '../lib/redux'

export default {
  component: InboxScreen,
  decorators: [(story: () => React.ReactNode) => <Provider store={store}>{story()}</Provider>],
  title: 'InboxScreen',
};

const Template:Story<InboxScreenArgs> = args => <PureInboxScreen {...args} />;

export const Default = Template.bind({});

export const Error = Template.bind({});
Error.args = {
  error: 'Something',
};
```

完整代碼可以參考[此處](https://github.com/marsen/intro-to-storybook-typescript)。

## 參考

- [Learn Storybook(javascript version)](https://www.learnstorybook.com/intro-to-storybook)
- [React+TypeScript Cheat sheets](https://github.com/typescript-cheatsheets/react)
- Component Library
  - [React](https://reactjs.org/)
  - [Vue](https://vuejs.org/)
  - [Angular](https://angular.io/)
- State Management Library
  - [Flux](https://facebook.github.io/flux/)
  - [Redux](https://redux.js.org/)
  - [Akita](https://datorama.github.io/akita/)

(fin)
