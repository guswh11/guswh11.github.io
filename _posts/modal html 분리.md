# modal html 분리 
## before 

productInfoView.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">

    <link rel="stylesheet"
          href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">

    <script src="https://code.jquery.com/jquery-3.2.1.min.js"></script>
    <script type="text/javascript" src="../static/js/getProductInfo.js"></script>
    <script type="text/javascript" src="../static/js/saveProductInfo.js"></script>

    <title>product Info</title>
    <style type=text/css>
        .productInfo
        {
            padding: 1%;
        }
        .projectNames
        {
            float: left;
            padding: 1%;
        }
        .projectUrls
        {
            float: left;
            padding: 1%;
        }
        .modal {
            display: none; /* Hidden by default */
            position: fixed; /* Stay in place */
            margin: 10% auto; /* 15% from the top and centered */
            z-index: 1; /* Sit on top */
            width: 80%;
            overflow: auto; /* Enable scroll if needed */
        }
    </style>
</head>
<body>
<div class="productInfo">
    <button id="loadProductBtn" type="button" class="btn btn-info btn-sm">프로젝트 불러오기</button>
    <button id="addProductBtn" type="button" class="btn btn-info btn-sm"
            data-toggle="modal" >새 프로젝트 등록</button>
    <hr/>
    <p> product id: </p><div id="productId"></div><hr/>
    <p> product name: </p><div id="productName"></div><hr/>
    <p> registered date: </p><div id="regDate"></div><hr/>
    <p> product summary: </p><div id="productSummary"></div><hr/>
    <div style="clear: both;">
        <p> product tags: <span id="productTags"></span></p>
    </div>
    <hr/>
    <div class="includedProjects">
        <p> included projects: </p>
        <div class="projectNames">
            <p>project name</p>
            <span id="projectName"></span>
        </div>
        <div class="projectUrls">
            <p>url</p>
            <span id="projectUrl"></span>
        </div>
    </div>
</div>

<!-- The Modal -->
<div id="saveProductModal" class="modal">
    <!-- Modal content -->
    <div class="modal-content">
        <div class="modal-content">
            <div class="modal-header">
                <h4 id="modal-title" class="modal-title">새 프로젝트 등록</h4>
            </div>
            <div class="modal-body">
                <table class="table">
                    <tr>
                        <td>프로덕트 제목</td>
                        <td><input class="form-control" id="projectNameInput" type="text"></td>
                    </tr>
                    <tr>
                        <td>프로덕트 설명</td>
                        <td><textarea class="form-control" id="projectSummaryInput" rows="10"></textarea></td>
                    </tr>
                    <tr>
                        <td>태그</td>
                        <td>
                            <textarea class="form-control" id="projectTagInput1" rows="1"></textarea>
                            <textarea class="form-control" id="projectTagInput2" rows="1"></textarea>
                            <textarea class="form-control" id="projectTagInput3" rows="1"></textarea>
                        </td>
                    </tr>
                </table>
            </div>
            <div class="modal-footer">
                <button id="submitBtn" type="button" class="btn btn-success">등록</button>
                <button id="closeBtn" type="button" class="btn btn-default" data-dismiss="modal">취소</button>
            </div>
        </div>
    </div>
</div>
</body>
</html>
```

getProductInfo.js
```javascript
$(document).ready(function(){
    $('#loadProductBtn').on('click', function(){
        $.ajax({
            url: "http://localhost:8080/product-info/get/3",
            type: "GET",
            success: function(data){
                showParsedResult(data);
            },
            error: function(){
                alert("error");
            }
        });
    });

    $('#addProductBtn').on('click', function(){
        $("#saveProductModal").show();
    });
});

function showParsedResult(data){
    $("#productId").html(data["productId"]);
    $('#productName').html(data["productName"]);
    $('#regDate').html(data["regDate"]);
    $('#productSummary').html(data["productSummary"]);
    $.each(data["productTags"], function(index, tag){
        $('#productTags').append(tag);
        if(index!==data["productTags"].length-1){
            $('#productTags').append(", ");
        }
    });
    //연관 프로젝트 get 필요 
}
```

saveProductInfo.js
```javascript
$(document).ready(function() {
    var url = '';
    var pid = 0;

    $("#submitBtn").click(function(){
        url = "http://localhost:8080/product-info/save";
        console.log("submit clicked");
        saveProjectInfo();
    });

    $("#closeBtn").click(function(){
        $("#saveProductModal").hide();
    });

    function saveProjectInfo() {
        var data = {
            "projectId" : 2,
            "projectName" : $("#projectNameInput").val(),
            "projectSummary" : $("#projectSummaryInput").val(),
            "projectTags" : [$("#projectTagInput1").val()],
            "projectMembers": [
                {
                    "userId": 3,
                    "roleId": 2
                }
            ],
            "projectToolAndAuth": [
                {
                    "toolId": 2,
                    "projectUrl": null,
                    "userId": "ID..2",
                    "userPw": "PW..2",
                    "apiKey": "API KEY..2",
                    "toolName": "Name..2"
                }
            ]
        };

        $.ajax({
            url: url,
            type: "POST",
            data: JSON.stringify(data),
            contentType: "application/json",
            success: function(data){
                console.log(data);
                alert("success");
            },
            error: function(){
                alert("error");
            }
        });
    }
});
```

## after
productInfoView.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">

    <link rel="stylesheet"
          href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">

    <script src="https://code.jquery.com/jquery-3.2.1.min.js"></script>
    <script type="text/javascript" src="../static/js/getProductInfo.js"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
    <title>product Info</title>
    <style type=text/css>
        .productInfo
        {
            padding: 1%;
        }
        .projectNames
        {
            float: left;
            padding: 1%;
        }
        .projectUrls
        {
            float: left;
            padding: 1%;
        }
        .modal-dialog {
            position: fixed; /* Stay in place */
            z-index: 1; /* Sit on top */
            width: 80%;
            overflow: auto; /* Enable scroll if needed */
        }
    </style>
</head>
<body>
<div id="body" class="productInfo">
    <button id="loadProductBtn" type="button" class="btn btn-info btn-sm">프로젝트 불러오기</button>

    <a data-toggle="modal" href="addProductInfoView.html" data-target="#addProductModal" role="button" data-backdrop="static">
        <span class="btn btn-info btn-sm">새 프로덕트 등록</span>
    </a>

    <hr/>
    <p> product id: </p><div id="productId"></div><hr/>
    <p> product name: </p><div id="productName"></div><hr/>
    <p> registered date: </p><div id="regDate"></div><hr/>
    <p> product summary: </p><div id="productSummary"></div><hr/>
    <div style="clear: both;">
        <p> product tags: <span id="productTags"></span></p>
    </div>
    <hr/>
    <div class="includedProjects">
        <p> included projects: </p>
        <div class="projectNames">
            <p>project name</p>
            <span id="projectName"></span>
        </div>
        <div class="projectUrls">
            <p>url</p>
            <span id="projectUrl"></span>
        </div>
    </div>
</div>

<div id="addProductModal" class="modal fade">
    <div class="modal-dialog">
        <div class="modal-content">
        </div>
    </div>
</div>

</body>
</html>
```

addProductInfoView.html
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <script src="https://code.jquery.com/jquery-3.2.1.min.js"></script>
    <script type="text/javascript" src="../static/js/saveProductInfo.js"></script>

</head>

<body>
<div class="modal-header">
    <h4 id="modal-title" class="modal-title">새 프로덕트 등록</h4>
</div>

<div class="modal-body">
    <table class="table">
        <tr>
            <td>프로덕트 제목</td>
            <td><input class="form-control" id="projectNameInput" type="text"></td>
        </tr>
        <tr>
            <td>프로덕트 설명</td>
            <td><textarea class="form-control" id="projectSummaryInput" rows="10"></textarea></td>
        </tr>
        <tr>
            <td>태그</td>
            <td>
                <textarea class="form-control" id="projectTagInput1" rows="1"></textarea>
                <textarea class="form-control" id="projectTagInput2" rows="1"></textarea>
                <textarea class="form-control" id="projectTagInput3" rows="1"></textarea>
            </td>
        </tr>
    </table>
</div>

<div class="modal-footer">
    <button id="submitBtn" type="button" class="btn btn-success">등록</button>
    <button id="closeBtn" type="button" class="btn btn-default" data-dismiss="modal">취소</button>
</div>
</body>
```

getProductInfo.js
```javascript
$(document).ready(function(){
    $('#loadProductBtn').on('click', function(){
        $.ajax({
            url: "http://localhost:8080/product-info/get/3",
            type: "GET",
            success: function(data){
                showParsedResult(data);
            },
            error: function(){
                alert("error");
            }
        });
    });
});

function showParsedResult(data){
    $("#productId").html(data["productId"]);
    $('#productName').html(data["productName"]);
    $('#regDate').html(data["regDate"]);
    $('#productSummary').html(data["productSummary"]);
    $.each(data["productTags"], function(index, tag){
        $('#productTags').append(tag);
        if(index!==data["productTags"].length-1){
            $('#productTags').append(", ");
        }
    });
    //연관 프로젝트 get 필요 
}
```

saveProductInfo.js
```javascript
$(document).ready(function() {
    var url = '';
    var pid = 0;

    $("#submitBtn").click(function(){
        url = "http://localhost:8080/product-info/save";
        console.log("submit clicked");
        saveProjectInfo();
    });

    function saveProjectInfo() {
        var data = {
            "projectId" : 2,
            "projectName" : $("#projectNameInput").val(),
            "projectSummary" : $("#projectSummaryInput").val(),
            "projectTags" : [$("#projectTagInput1").val()],
            "projectMembers": [
                {
                    "userId": 3,
                    "roleId": 2
                }
            ],
            "projectToolAndAuth": [
                {
                    "toolId": 2,
                    "projectUrl": null,
                    "userId": "ID..2",
                    "userPw": "PW..2",
                    "apiKey": "API KEY..2",
                    "toolName": "Name..2"
                }
            ]
        };

        $.ajax({
            url: url,
            type: "POST",
            data: JSON.stringify(data),
            contentType: "application/json",
            success: function(data){
                console.log(data);
                alert("success");
            },
            error: function(){
                alert("error");
            }
        });
    }
});
```

