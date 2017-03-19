---
layout: post
title:  "构建 RESTful 风格的 WCF 服务"
date:   2017-01-11
categories: C#
tags: [WCF, RESTful]
---

近期，公司项目有一个APP要做，数据访问打算用 WCF RESTful Service，但之前没有接触过，所有在实际开发的过程中踩了不少坑，特此记录。

### RESTful 简介

某度上是这么解释的：“REST（英文：Representational State Transfer，简称REST）描述了一个架构样式的网络系统，比如 web 应用程序。它首次出现在 2000 年 Roy Fielding 的博士论文中，他是 HTTP 规范的主要编写者之一。在目前主流的三种 Web 服务交互方案中，REST 相比于 SOAP（Simple Object Access protocol，简单对象访问协议）以及XML-RPC 更加简单明了，无论是对URL的处理还是对 Payload 的编码，REST 都倾向于用更加简单轻量的方法设计和实现。值得注意的是 REST 并没有一个明确的标准，而更像是一种设计的风格。”

### 定义 Book 实体

定义一个 Book 实体，只包含ID、名称和价格三个属性，具体代码如下：

``` cs
[DataContract]
public class Book
{
    [DataMember]
    public int Id { get; set; }

    [DataMember]
    public string Name { get; set; }

    [DataMember]
    public decimal Price { get; set; }
}
```


### 定义契约
定义契约的方式跟 WCF 一样，只是在每个具体的方法头部加了 WebGet 或者 WebInvoke。在这里，新增、修改、删除方法的返回类型都是集合，主要是便于展示数据。

> "*`Method`：* 获取或设置服务操作响应的协议 (如 http ) 方法；<br />
> *`UriTemplate`：* 用于服务操作的统一资源标识符 (URI) 模板；<br />
> *`ResponseFormat`：* 设置服务操作f啊出的响应的格式，这是一个枚举，有 Xml 和 Json 两个值，默认是 Xml，也就是说，如果未制定，默认返回的数据是 Xml 格式；<br />
> *`RequestFormat`：* 设置服务操作发出请求的格式，这也是一个枚举，同样有 Xml 和 Json 两个只，默认是 Xml；<br />
> *`BodyStyle`：* 获取和设置传入与传出服务操作的消息的正文样式。这同样是一个枚举。
> |类型|说明|
> |-|-|
> |WebMessageBodyStyle.Base | 不包装请求和响应|
> | WebMessageBodyStyle.Wrapped | 包装请求和响应|
> |WebMessageBodyStyle.WrappedRequest| 包装请求，但不包装响应|
> |WebMessageBodyStyle.WrappedResponse| 包装响应，但不包装请求|

``` cs
[ServiceContract]
public interface IBookService
{
    [WebGet(UriTemplate = "book/list", ResponseFormat = WebMessageFormat.Json)]
    [OperationContract]
    List<Book> GetList();

    [WebGet(UriTemplate = "book/list/{id}", ResponseFormat = WebMessageFormat.Json)]
    [OperationContract]
    Book GetById(string id);
    
    [WebInvoke(
        UriTemplate = "book/add", 
        Method = "POST", 
        RequestFormat = WebMessageFormat.Json, 
        ResponseFormat = WebMessageFormat.Json)]
    [OperationContract]
    List<Book> Add(Book model);


    [WebInvoke(
        UriTemplate = "book/update", 
        Method = "PUT", 
        RequestFormat = WebMessageFormat.Json, 
        ResponseFormat = WebMessageFormat.Json)]
    [OperationContract]
    List<Book> Update(Book book);

    [WebInvoke(
        UriTemplate = "book/delete/{id}", 
        Method = "DELETE", 
        ResponseFormat = WebMessageFormat.Json)]
    [OperationContract]
    List<Book> Delete(string id);
}
```
### 服务实现
服务实现通过操作一个集合来展示增删改查功能。
``` cs
public class BookService : IBookService
{
    private List<Book> _dataSource = new List<Book>()
    {
        new Book() {Id = 1, Name = "西游记", Price = 23},
        new Book() {Id = 2, Name = "红楼梦", Price = 22},
        new Book() {Id = 3, Name = "封神榜", Price = 23},
        new Book() {Id = 4, Name = "金瓶梅", Price = 33},
        new Book() {Id = 5, Name = "水浒传", Price = 25}
    }; 


    public List<Book> GetList()
    {
        return _dataSource;
    }

    public Book GetById(string id)
    {
        int idValue = int.TryParse(id, out idValue) ? idValue : 1;
        return _dataSource.FirstOrDefault(p => p.Id == idValue);
    }

    public List<Book> Add(Book model)
    {
        if (model == null)
        {
            model = new Book();
        }
        model.Id = _dataSource.Max(p => p.Id) + 1;
        _dataSource.Add(model);

        return _dataSource;
    }

    public List<Book> Update(Book model)
    {
        if (model == null)
            return _dataSource;
        var book = _dataSource.FirstOrDefault(p => p.Id == model.Id);
        if (book == null)
            return _dataSource;
        book.Name = model.Name;
        book.Price = model.Price;

        return _dataSource;
    }

    public List<Book> Delete(string id)
    {
        int idValue = int.TryParse(id, out idValue) ? idValue : 1;
        var model = _dataSource.FirstOrDefault(p => p.Id == idValue);
        if (model == null)
            return _dataSource;
        _dataSource.Remove(model);
        return _dataSource;
    } 
}
```
### Config 配置
定义了契约和实现了服务之后，还需要添加 Config 配置：
``` xml
<system.serviceModel>
    <behaviors>
        <serviceBehaviors>
        <behavior name="">
            <serviceMetadata httpGetEnabled="true" httpsGetEnabled="true" />
            <serviceDebug includeExceptionDetailInFaults="false" />
        </behavior>
        </serviceBehaviors>
        <endpointBehaviors>
        <behavior name="webHttp">
            <webHttp helpEnabled="true"/>
            <dataContractSerializer maxItemsInObjectGraph="2147483647"/>
        </behavior>
        </endpointBehaviors>
    </behaviors>
    <serviceHostingEnvironment aspNetCompatibilityEnabled="true" multipleSiteBindingsEnabled="true" />
    <services>
        <service name="WcfRestful.BookService">
        <endpoint address="" binding="webHttpBinding" behaviorConfiguration="webHttp" contract="WcfRestful.IBookService"/>
        </service>
    </services>
</system.serviceModel>
```

### 前端调用
``` javascript
$(document).ready(function () {
    $('#btnGet').click(function () {
        $.ajax({
            type: 'get',
            dataType: 'json',
            url: 'http://localhost:65416/BookService.svc/book/list',
            contentType: 'application/json;charest=utf-8',
            success: function (result) {
                createTable(result);
            }
        });
    });
});

function getInfo(id) {
    $.ajax({
        type: 'get',
        dataType: 'json',
        url: 'http://localhost:65416/BookService.svc/book/list/' + id,
        contentType: 'application/json;charest=utf-8',
        //data: { id: id },
        success: function (result) {
            var $ul = $('#bookInfo').empty();
            if (typeof (result) === "object") {
                $ul.append($('<li class="list-group-item">ID：' + result.Id + '</li>'))
                    .append($('<li class="list-group-item">名称：' + result.Name + '</li>'))
                    .append($('<li class="list-group-item">价格：' + result.Price + '</li>'));
            }
        }
    });
}

function getById(id) {
    $.ajax({
        type: 'get',
        dataType: 'json',
        url: 'http://localhost:65416/BookService.svc/book/list/' + id,
        contentType: 'application/json;charest=utf-8',
        success: function (result) {
            var $ul = $('#bookInfo').empty();
            if (typeof (result) === "object") {
                $('#Edit #Id').val(result.Id);
                $('#Edit #Name').val(result.Name);
                $('#Edit #Price').val(result.Price);
            }
        }
    });
}

function createTable(result) {
    var $tbody = $('#tBody').empty();
    if (typeof (result) === "object" && result.length > 0) {
        $.each(result, function (index, item) {
            var $tr = $('<tr></tr>')
                .append('<td>' + item.Id + '</td>').append('<td>' + item.Name + '</td>')
                .append('<td>' + item.Price + '</td>').append('<td><a class="btn btn-info" href="javascript:void(0)" onclick="getInfo(' + item.Id + ')">详细</a> <a class="btn btn-warning" href="javascript:void(0)" onclick="getById(' + item.Id + ')">修改</a> <a class="btn btn-danger" href="javascript:void(0)" onclick="Delete(' + item.Id + ')">删除</a></td>');
            $tbody.append($tr);
        });
    }
}

function add() {
    var name = $('#add #Name').val() || '',
        price = $('#add #Price').val() || '';

    if (name === '') {
        $('#add .alert').removeClass('alert-success').addClass('alert-warning').html('请填写名称');
        return;
    }
    if (price === '') {
        $('#add .alert').removeClass('alert-success').addClass('alert-warning').html('请填写价格');
        return;
    }

    $.ajax({
        type: 'POST',
        dataType: 'json',
        url: 'http://localhost:65416/BookService.svc/book/add',
        contentType: 'application/json;charest=utf-8',
        data: JSON.stringify({ Name: name, Price: price }),
        success: function (result) {
            createTable(result);
            $('#add .alert').removeClass('alert-warning').addClass('alert-success').html('新增成功');
        }
    });
}

function edit() {
    var id = $('#Edit #Id').val() || '',
        name = $('#Edit #Name').val() || '',
        price = $('#Edit #Price').val() || '';

    if (id === '') {
        $('#Edit .alert').removeClass('alert-warning').addClass('alert-warning').html('请选择要修改的内容');
        return;
    }
    if (name === '') {
        $('#Edit .alert').removeClass('alert-warning').addClass('alert-warning').html('请填写名称');
        return;
    } 
    if (price === '') {
        $('#Edit .alert').removeClass('alert-warning').addClass('alert-warning').html('请填写价格');
        return;
    }

    $.ajax({
        type: 'PUT',
        dataType: 'json',
        url: 'http://localhost:65416/BookService.svc/book/update',
        contentType: 'application/json;charest=utf-8',
        data: JSON.stringify({ Id:id, Name: name, Price: price }),
        success: function (result) {
            createTable(result);
            $('#Edit .alert').removeClass('alert-warning').addClass('alert-success').html('修改成功');
        }
    });
}

function Delete(id) {
    $.ajax({
        type: 'DELETE',
        dataType: 'json',
        url: 'http://localhost:65416/BookService.svc/book/delete/' + id,
        contentType: 'application/json;charest=utf-8',
        success: function (result) {
            createTable(result);
        }
    });
}
```

### 使用中遇到的问题

一个简单完整的例子就完了。在实际的过程中，有以下需要注意的地方：

1. 如果指定了 ```BodyStyle = WebMessageBodyStyle.Wrapped``` 或 ```BodyStyle = WebMessageBodyStyle.WrappedResponse```，返回的数据会被封装，例如：
    ``` json
    {"GetListResult":[
        {"Id":1,"Name":"西游记","Price":23},
        {"Id":2,"Name":"红楼梦","Price":22},
        {"Id":3,"Name":"封神榜","Price":23},
        {"Id":4,"Name":"金瓶梅","Price":33},
        {"Id":5,"Name":"水浒传","Price":25}]
    }
    ```
    那么，前台在解析的时候就需要做特殊处理；
2. 对于 Get 请求的参数传入，可以有以下几种方式：
    - 指定 UriTemplate 为 ```UriTemplate = "book/list/{id}"```，后台可以一个参数或者是一组 json 格式序列化之后的字符串；
    - 指定 UriTemplate 为 ```UriTemplate = "book/list/{id}/{name}"```，后台可以接收对应的两个参数；
    - 对于 Get ， 如果前台有多个参数提交，可以先将采用先序列化,；
        ``` cs
        后台代码：
        [WebGet(
            UriTemplate = "book/list/?query={query}", 
            ResponseFormat = WebMessageFormat.Json)]
        [OperationContract]
        List<Book> GetList(string query);
        ```
        ``` javascript
        前台代码：
        $.ajax({
            type: 'get',
            dataType: 'json',
            url: 'http://localhost:65416/BookService.svc/book/list/',
            contentType: 'application/json;charest=utf-8',
            data: { query: JSON.stringify({ Name: name, Id: id }) },
            success: function (result) {
                // todo;
            }
        });
        ```
    - 如果是 Post 提交，后台对应的服务的参数是对象，前台在调用的时候需要先将提交的数据进行序列化。

源代码：[https://github.com/wmgwugang/WcfRestful.git](https://github.com/wmgwugang/WcfRestful.git)

