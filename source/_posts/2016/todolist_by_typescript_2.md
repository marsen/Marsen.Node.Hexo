---
title: '[KATA]用 typescript 作一個簡易的 TodoList(二) - 用JQuery實作'
date: 2016/10/14 10:34:27
tag: 
- TypeScript
- BootStrap
- NPM
- JQuery
---
## 設計理念

1. 顯示/新增/刪除 TodoList
2. TodoList 會是一堆todoItem的集合,所以要定義todoItem的形別
    - Content : todoItem 的內容
    - Status : todoItem 的狀態,完成(done)、未完成(undo) ,設計成列舉
3. 主要的功能
   - 建立todoItem
   - 完成todoItem
   - 繪製todoList到前端的畫面上

## 自我分析

跟 UI 耦合太高,Render 應該與 TodoService 分離 ,
DOM 註冊事件相依在 Service 裡面要抽離也不好抽 。
沒有先寫測試 , 要想一想怎麼與 UI 層作隔離。

## 程式碼

### 建立 BaseService

```typescript
    class BaseService<T>{
        constructor(private type: string) {}
        
        List:Array<T> ;

        Create(data: T) {
            this.List.push(data);
        }

        Delete(data: T){
            var index = this.List.indexOf(data);
            this.List.splice(index,1) ;
        }

        Render(){

        }
    }

```

### 建立 todoItem interface

```typescript
interface todoItem {
    Content: string;
    Status: todoStatus;
}

```

### 建立 todoItem Status 列舉

```typescript
enum todoStatus{
    undo,
    done,    
}

```

### 建立 TodoService

```typescript
class TodoService extends BaseService<todoItem>{
    constructor(){
        super("todoItem");        
    }

    public Render() : void {
        let doneHtml = '';
        let undoHtml = '';
        
        this.List.forEach((item)=>{
            if(item.Status == todoStatus.done){
                doneHtml += `<li>${item.Content}<button class="recover-item btn btn-default btn-xs pull-right"><span class="glyphicon glyphicon-share-alt"></span></button><button class="remove-item btn btn-default btn-xs pull-right"><span class="glyphicon glyphicon-remove"></span></button></li>`;
            } else if (item.Status == todoStatus.undo){
                undoHtml += `<li class="ui-state-default"><div class="checkbox"><label><input type="checkbox" value="" />${item.Content}</label></div></li>` ;
            }
        });
        $("#done-items").html(doneHtml);
        $('#sortable').html(undoHtml);
        $('.add-todo').val('');
    }

    Delete(data: todoItem){
        data.Status = todoStatus.done ;
    }



    Init(){
        //// todoList in localStorage         
        var list = window.localStorage.getItem("todoList");        
        if(!list){
            this.List = new Array<todoItem>();
        }else{
            this.List = JSON.parse(list);
        }
        window.onbeforeunload = (evt) => {
            window.localStorage.setItem("todoList",JSON.stringify(this.List));
        };

        // mark task as done
        $('.todolist').on('change','#sortable li input[type="checkbox"]',(evt)=>{
            var self = evt.target;
            var text = $(self).parent().text();
            if($(self).prop('checked')){
                var doneItem = this.List.filter((i)=>{return text == i.Content;})[0];
                this.Delete(doneItem);
                this.Render();
            }
        });

        $('.add-todo').on('keypress',(evt) => {
            evt.preventDefault
            if (evt.which == 13) {
                if($(evt.target).val() != ''){
                    var todo = $(evt.target).val();
                    this.Create(  {
                        Content : $(evt.target).val() ,
                        Status : todoStatus.undo 
                    });
                    this.Render();                 
                }else{
                    // some validation
                }
            }
        });
        
        $('.todolist').on('click', '#done-items li button.recover-item',(evt)=>{
            var text = $(evt.target).parent().parent().text();
            var recoverItem = this.List.filter((i)=>{return text == i.Content;})[0];
            recoverItem.Status = todoStatus.undo ;
            this.Render();
        });

        $('.todolist').on('click','#done-items li button.remove-item' ,(evt)=>{
            var text = $(evt.target).parent().parent().text();
            var removeItem = this.List.filter((i)=>{return text == i.Content;})[0];
            var index = this.List.indexOf(removeItem);
            this.List.splice(index,1) ;
            this.Render();
        });

        $('#checkAll').on('click',(evt)=>{
            this.List.forEach((item)=>{
                item.Status = todoStatus.done;
            });
            this.Render();
        });

        //// Render
        this.Render();
    }
}

```

### 使用建立好的 TodoService

```typescript
var todoService = new TodoService();
todoService.Init();
 
```

(fin)
