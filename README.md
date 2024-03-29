### 1. BaseAction
- Việc thực hiện một hành động như `openModal`, hoặc get dữ liệu, ... nên được implement ở 1 chỗ, việc tái sử dụng sẽ hiệu quả hơn
Ví dụ: open một modal bệnh án thì mỗi trang sẽ có cách hiển thị 'nút open bệnh án' khác nhau (button hoặc thẻ a), nhưng việc gọi `this.openModal` nên được tập trung 1 chỗ
```jsx
export default class BaseAction extends BaseConsumer {
    static propTypes = {
        renderContent: PropTypes.func.isRequired
    } 
    onAction() {
        throw "not implemented onAction"
    }
    consumerContent() {
        const { renderContent } = this.props;
        return renderContent(this.onAction);
    }
}
```
- Giả sử cần một component để mở bệnh án, override hàm `onAction` 
```jsx
export default class BenhAnModalCreator extends BaseAction {
    onAction(){
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

### 2. Các phương thức chỉnh sửa state và một vài khái niệm mới
Sẽ có 2 khái niệm mới là `StateManager` và `StateWrapper`. `StateWrapper` sẽ được sử dụng nhiều như consumer, trong khi `StateManager` chỉ sử dụng ở base phía trên (tức CleanNode)

- `StateManager`: đây là một Component cung cấp các phương thức chỉnh sửa lên `state`, tên cụ thể các phương thức có thể lên tận file xem, VD một số như: `this.updateLocalObject`, `this.addLocalElement`, thực chất cách đặt tên này giống đặt tên của `BasePage` cũ và thêm chữ "Local" thêm vào sau động từ, chẳng hạn như "update" thì thành "updateLocal"
Các phương thức này dùng để chỉnh sửa `state` của chính nó, lưu ý là CỦA CHÍNH NÓ, VD:
```jsx
class Consumer extends StateManager {
    state = {
        data: {
            value: 1,
        }
    }
    _onChange = e => {
        // object được update phải là object có reference (trực tiếp hoặc gián tiếp) tới state của chính nó
        this.updateLocalObject(this.state.data, {value: e.target.value});
    }
}
```

- `BasePage` và `BaseConsumer` bản chất đều kế thừa từ `StateManager` vì vậy các component kế thừa 2 class này đều sở hữu các phương thức của `StateManager`, VD:
```jsx
class Person extends BaseConsumer {
    constructor(props){
        super(props);
        this.state = {
            user: {
                age: 1,
            }
        }
    }
    _plus = () => {
        this.updateLocalObject(this.state.user, {age: this.state.user.age + 1});
    }
}
```

- Các phương thức kiểu `updateObject` thực chất là các phương thức của `StateManager` nhưng được đổi tên bằng cách remove chữ "Local" sau đó feed vào Context từ `BasePage` (khái niệm React Context sẽ được giải thích sau hoặc ai chưa biết thì tự tìm hiểu trước) nên ở `BaseConsumer` có các hàm này nhưng chỉ update được các object có reference từ `state` của `BasePage` là vì vậy.

- `StateWrapper`: đây thực chất là component kế thừa từ `BaseConsumer`, có 2 phương thức là `setData` và `getData` đồng thời cần implement `wrapperContent` để render, cách sử dụng tương tự như `BaseRouteWrapper`, VD:
```jsx
class Product extends StateWrapper {
    componentDidMount(){
        let X = {
            data: {
                name: "X-men",
                price: 10000,
            }
        }; //data lấy từ ajax về hoặc data dummy
        this.setData(X);
    }
    wrapperContent(){
        const product = this.getData(); // lấy là X phía trên
        return (
            <div>
                Product 
                <ProductHeader 
                    data={product.data}
                />
            </div>
        )
    }
}
class ProductHeader extends BaseConsumer {
    consumerContent(){
        const {data} = this.props;
        return (
            <div>
                Name: {data.name}
                <input 
                    placeholder="Change price..."
                    value={data.price}
                    onChange={e => {
                        this.updateObject(data, {price: e.target.value});
                    }}

                />
            </div>
        )
    }
}
```
Như VD trên, ở did mount của `Product` sử dụng `setData` để tạo ra data, gắn vào state của `BasePage`, vì vậy ở `ProductHeader` có thể update được data đó
Lưu ý: hàm `setData` có bản chất tương tự hàm `setState`, vì vậy vị trí sử dụng hàm này tương tự `setState` tức không được sử dụng ở một số hàm như `render`, `constructor`, `componentDidUpdate`, ....

### 3. Lưu ý về Clone
Khái niệm này được sử dụng khi cần render một component, component này nhận props từ bên ngoài và update các props đó nhưng chúng ta không muốn các props đó bị thay đổi ở bên ngoài component đó.

### 4. Không được khai báo bằng `var`, khai báo bằng `let` và `const` 

### 5. DisplayName
Các component cần có thêm biến static displayName là các component kế thừa StateWrapper, nếu không kế thừa thì không cần
```jsx
import StateWrapper from 'BaseComponent/StateWrapper';
class Item extends StateWrapper {
    // code
}
Item.displayName = "OrAbcXyzWhateverAsLongAsIt'sUnique"
```
### 6. Ajax Post/Get
Hai phương thức ajax get và ajax post đã được đưa lên BasePage nhằm đơn giản hóa cách sử dụng, cụ thể:
```jsx
// hàm get là this.ajaxGet
this.ajaxPost({ 
    url: `/api/ControllerName/MethodName`,
    data: data,
    success: ack => {
        // thay thế successCallback ở bản cũ
    },
    unsuccess: ack => {
        // thay thế unsuccessFunction ở bản cũ
    },
    error: (xhr, status, err) => {
        // thay thế errorCallback ở bản cũ
    },
    unAuthorized: () => {
        // được invoke khi user không có quyền (error code: 401)
    }
})
```
- Thông thường, ở unsuccess và error thì chúng ta thường báo lỗi lên UI cho người dùng nhìn thấy, tuy nhiên việc làm này thường không được làm hoặc quên bởi dev, vì vậy nếu 2 hàm này không được implement ở parameter thì 2 hàm này vẫn tự động log ra error lên UI (muốn hiểu rõ thì có thể lên BasePage xem hàm _sendAjax)
- Default thì `unAuthorized` sẽ invoke hàm `this.onUnauthorized`, hàm sẽ có thể (nên) được override lại ở Root

```jsx
this.ajaxPost({
    url: `/api/ControllerName/MethodName`,
    data: data,
    success: ack => {
        // thay thế successCallback ở bản cũ
    }
})
```

Lưu ý: `data` ở `ajaxPost` từ giờ sẽ ko được `JSON.stringify` trước, mà để nguyên "cục" `data` vào. Cụ thể là `data` sẽ được stringify ở trên `BasePage` bằng hàm `this.JSONStringify`, hàm này có thể override lại ở Root nếu muốn customize lại cách stringify

### 7. PopoverManager
Dùng mở một [Popper](https://material-ui.com/components/popper/)
```jsx
_testPopup = (e) => {
	let target = e.currentTarget
	let popupFunc = () => ({
		body: (
			<TestPopover data={this.props.data.testPopoverData} />
		),
		anchorEl: target,
		clickAwayClose: true,
            	placement: "bottom-end"
	})
	this.openPopup(popupFunc);
}

render(){
	return (
		<div>
			<span onClick={this._testPopover}>click vào đây để mở popover</span>
		</div>
	)
}
```
Hàm `openPopup(popoverFunction: object) : string` trả về id của popup (dùng để tắt) và nhận vào parameter là `popupFunc` là function trả về object với shape như sau:
Property name | Type | Default | Description
:--- | :--- | :--- | :---
`body` | node | | tương tự body của modal function
`anchorEl` | element | | "cái neo" để popup "bám" vào
`onCloseCallback` | func | | hàm được gọi sau khi popup tắt
`clickAwayClose` | boolean | false | `true` nếu muốn popover tự tắt khi click ra ngoài hoặc nhấn ESC
`placement` | string | "bottom" | xem [Prop placement](https://material-ui.com/api/popper/#props) của Popper

Hàm `closePopup(popupId?: null, callback?: null)` có parameter là `popupId` là `id` ở popup trả về ở hàm `openPopup`

Ngoài ra, component truyền vào `body` ở `popupFunc` sẽ tự động nhận được 1 prop là `popupId`.

### 8. DrawerManager
Dùng mở một [Drawer](https://material-ui.com/components/drawers/)
```jsx
_testDrawer = () => {
	let drawerFunc = () => ({
		body: <TestDrawer data={this.props.data.testPopoverData} />
	})
	this.openDrawer(drawerFunc);
}

render(){
	return (
		<div>
			<span onClick={this._testDrawer}>click vào đây để mở drawer</span>
		</div>
	)
}
```
Hàm `openDrawer(drawerFunction: object) : string` trả về id của drawer (dùng để tắt) và nhận vào parameter là `drawerFunction` là function trả về object với shape như sau:
Property name | Type | Default | Description
:--- | :--- | :--- | :---
`body` | node | | tương tự body của modal function

Hàm `closeDrawer(drawerId?: null, callback?: null)` có parameter là `drawerId` là id ở drawer trả về ở hàm `openDrawer`

Ngoài ra, component truyền vào `body` ở `drawerFunction` sẽ tự động nhận được 1 prop là `drawerId`.


