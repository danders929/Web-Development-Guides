##Create Vite Project
  - npm create vite@latest
  - React -> Javascript + SWC
  - name folder
  - cd (name of folder)
  - npm i
  - delete unused code in App.jsx
  - npm run dev to start local server

##Make React Components
  - Create components ****Components start with an uppercase letter and .jsx extension****
  - Import and call your component in the App.jsx return statement
```jsx
import TaskList from './TaskList'
import './App.css'

function App() { 
  return (
    <>
      <TaskList />
    </>
  )
} 

export default App;
```
  - Import and call your components that reference each other
```jsx
  import Task from './Task';
import './App.css' 

function TaskList() {
  return (
    <>
      <Task />
    </>
  )
}  

export default TaskList;
```

  - Create any CSS styling as needed
