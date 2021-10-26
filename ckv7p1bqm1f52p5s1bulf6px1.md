## How to use React Web Components in Angular

Howdy folks!! In the previous article we learned  [How to use Angular Web Components in React](https://thalava.com/how-to-use-angular-web-components-in-react-js) . In this tutorial we'll learn how to use A React component in an Angular Project 😎

We'll start with installing our angular project first then slowly install React dependencies and integrate our Components

#### 1. Install Angular CLI if you don't have it already
``` 
npm install -g @angular/cli
```
#### 2. Create a fresh new Angular Project
```
ng new my-angular-app
```
#### 3. Add `react` and `react-dom` as dev dependency in your project
```
npm i --save -D @types/react @types/react-dom
```
You should find these new dependencies added to your project. The version can be a bit upgraded by the time you find this article. Feel free to leave a comment, I'll update this tutorial 😉
```
"dependencies": {
    ...
    "react": "^17.0.2",
    "react-dom": "^17.0.2",
    ...
  },
"devDependencies": {
    ...
    "@types/react": "^17.0.2",
    "@types/react-dom": "^17.0.2",
    ...
}
```
We need to add the following property in `tsconfig.json` in our Angular project for typescript to recognize and compile React components
```
{
     ... 
     "jsx": "react",
     ...
}
```
#### 4. Create our React component at the src directory of our angular project
`CustomReactComponent.tsx`
```
import * as React from 'react';
import { FunctionComponent, useEffect, useRef, useState } from 'react';

export interface IMyComponentProps {
  counter: number;
  onClick?: () => void;
}

export const CustomReactComponent: FunctionComponent<IMyComponentProps> = (props: IMyComponentProps) => {

  const timerHandle = useRef<number | null>(null);
  const [stateCounter, setStateCounter] = useState(42);

  useEffect(() => {
    timerHandle.current = +setInterval(() => {
      setStateCounter(stateCounter + 1);
    }, 2500);

    return () => {
      if (timerHandle.current) {
        clearInterval(timerHandle.current);
        timerHandle.current = null;
      }
    };
  });

  const {counter: propsCounter, onClick} = props;

  const handleClick = () => {
    if (onClick) {
      onClick();
    }
  };

  return (
    <div>
        <div>Props counter: {propsCounter}
          <button type="button" onClick={handleClick}>click to increase</button>
        </div>
        <div>State counter: {stateCounter}</div>
    </div>
  );
};
```
Here stateCounter will trigger 2.5 sec after when the component loads and I define a function handleClick which I will operate through my Angular project.

#### 5. Add a wrapper component to generate the web component
`CustomReactComponentWrapper.tsx`
```
import {
    AfterViewInit,
    Component,
    ElementRef,
    EventEmitter,
    Input,
    OnChanges,
    OnDestroy,
    Output,
    SimpleChanges,
    ViewChild,
    ViewEncapsulation
} from '@angular/core';
import * as React from 'react';
import * as ReactDOM from 'react-dom';
import { CustomReactComponent } from './CustomReactComponent';


const containerElementName = 'customReactComponentContainer';

@Component({
    selector: 'app-my-component',
    template: `<span #${containerElementName}></span>`,
    // styleUrls: [''],
    encapsulation: ViewEncapsulation.None,
})
export class CustomReactComponentWrapperComponent implements OnChanges, OnDestroy, AfterViewInit {
    @ViewChild(containerElementName, { static: true }) containerRef!: ElementRef;
    
    @Input() public counter = 10;
    @Output() public componentClick = new EventEmitter<void>();

    constructor() {
        this.handleDivClicked = this.handleDivClicked.bind(this);
    }

    public handleDivClicked() {
        if (this.componentClick) {
            this.componentClick.emit();
            this.render();
        }
    }

    ngOnChanges(changes: SimpleChanges): void {
        this.render();
    }

    ngAfterViewInit() {
        this.render();
    }

    ngOnDestroy() {
        ReactDOM.unmountComponentAtNode(this.containerRef.nativeElement);
    }

    private render() {
        const { counter } = this;

        ReactDOM.render(
            <React.StrictMode>
                <div>
                    <CustomReactComponent counter={counter} onClick={this.handleDivClicked} />
                </div>
            </React.StrictMode>
            , this.containerRef.nativeElement);
    }
}
```
Here I wrap the React component within a Angular Wrapper Component and generate the web component which I will use at my Angular project to generate the React instance.

#### 6. Add the React web component to our angular web project
I am adding the component at `app.component.html` for the sake of this tutorial, while working on an actual project you can use the React component within any Angular component in your entire application.
`app.component.html`
```
<div id="myReactComponentContainer">
  <app-my-component [counter]="counter" (componentClick)="handleOnClick($event)"></app-my-component>
</div>
```

#### 7. Now add the business logic and events at your `app.component.ts` and your are all set.. 😁
```
export class AppComponent {
  ...
  public counter = 21;

  public handleOnClick(stateCounter: number) {
    this.counter++;
  }
}
```

![giphy.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1635225855039/vIOYOogMT.gif)
![1633098478.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1635225999089/vBGZKaD2L.gif)