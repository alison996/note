#### React 类式组件生命周期

##### 1.首次加载时
###### 1) contructor
类式组件的构造器，可在构造器中初始化组件状态( state )

###### 2) getDerivedStateFromProps
state的值在`任何时候`都取决于props( props变化时更新state )，接收两个参数`props`、`state`

###### 3) render
渲染函数，返回vnode

###### 4) componentDidMount
组件`第一次`被渲染到DOM中，可在此钩子中设置定时器，调用AJAX


##### 更新时
###### 1) getDerivedStateFromProps

###### 2) shouldComponentUpdata
根据return的`布尔值`决定是否更新组件

###### 3) render

###### 4) getSnapshotBeforeUpdate
获取更新前的快照，其返回值默认作为`componentDidUpdate`第三个参数

###### 5) componentDidUpdate
组件更新之后，其可接收三个参数`未更新前的props`、`未更新前的state`、`getSnapshotBeforeUpdate的返回值`

##### 卸载时
###### 1) componentWillUnmount
组件即将卸载之前调用的钩子，可在此阶段销毁定时器

##### Demo
```javascript
import { Component, Fragment } from 'react';
import { render, unmountComponentAtNode } from 'react-dom';

class Demo extends Component {
  constructor(props) {
    super(props);
    this.state = { title: 'This is a initial demo.' };
    console.log('constructor');
  }

  static getDerivedStateFromProps(props, state) {
    console.log('getDerivedStateFromProps');
    return { subTitle: props.subTitle };
  }

  componentDidMount() {
    console.log('componentDidMount');
  }

  getSnapshotBeforeUpdate() {
    console.log('getSnapshotBeforeUpdate');
    return { snapshotRes: 'getSnapshotBeforeUpdate' };
  }

  shouldComponentUpdate() {
    console.log('shouldComponentUpdate');
    return true;
  }

  componentDidUpdate(preProps,preState,snapshotValue) {
    console.log('componentDidUpdate');
  }

  componentWillUnmount() {
    console.log('componentWillUnmount');
  }

  /**
	 * 使用箭头函数，防止找不到this
	 */
	updateComponent = () => {
		this.forceUpdate();
	}

  unmountComponent = () => {
    unmountComponentAtNode(document.querySelector('#app'));
  }

  render() {
    console.log('render');
    const { title, subTitle } = this.state;
    return (
      <Fragment>
        <h1>{ title }</h1>
        <h2>{ suTitle }</h2>
        <button onClick={ this.updateComponent }>Force update</button>
        <button onClick={ this.unmountComponent }>Unmount component</button>
      </Fragment>
    )
  }
}

render(<Demo subTitle="Alison" />, document.querySelector('#app'));
```