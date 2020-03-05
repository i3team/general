##### 1. BaseAction
- Việc thực hiện một hành động như `openModal`, hoặc get dữ liệu, ... nên được implement ở 1 chỗ, việc tái sử dụng sẽ hiệu quả hơn
Ví dụ: open một modal bệnh án thì mỗi trang sẽ có cách hiển thị 'nút open bệnh án' khác nhau (button hoặc thẻ a), nhưng việc gọi `this.openModal` nên được tập trung 1 chỗ
```jsx
export default class BaseAction extends BaseConsumer {
    static propTypes = {
        renderContent: PropTypes.func.isRequired
    }
    onClick() {
        throw "not implemented onClick"
    }
    consumerContent() {
        const { renderContent } = this.props;
        return renderContent(this.onClick);
    }
}
```
- Giả sử cần một component để mở bệnh án, implement như sau 
```jsx
export default class BenhAnModalCreator extends BaseAction {
    onClick() {
        this.openModal(bla bla bla);
    }
}
```
- Cách sử dụng:
```jsx
<BenhAnModalCreator
    renderContent={(onClickFunc) => {
        return (
            <div onClick={onClickFunc}>
                nhấn vào đây để mở bệnh án
            </div>
        )
    }}
/>
```
