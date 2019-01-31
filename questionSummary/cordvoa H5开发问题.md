# cordvoa H5开发问题

### 1.关于jquery封装 ajax 进行文件下载空白  
***
##### 1-1 问题描述 ：
  后端返回数据流，前端使用blob与a链接的download实现文件下载，出现下载文件打开空白
##### 1-2  问题原因 ：
  jquery在封装ajax时，对返回数据进行了处理，默认将 **流数据** 转成了 **string类型** 数据，使得，前端在接收到数据，进行下载时，出现空白
  
##### 1-3  解决方法 ：
  自己封装ajax请求，防止数据类型被修改 *(适用于PC端的流文件下载)*  

  ```
    var url = "/electronicPolicy/downLoadPolicy";
    var xhr = new XMLHttpRequest();
    var data = JSON.stringify({
      policySort: policySort,
      policyNo: policyNo,
      credentialNo: credentialNo,
      identifyCode: identifyCode,
    });
    xhr.open("POST", url, true); // 也可以使用POST方式，根据接口
    xhr.setRequestHeader("Content-Type", "application/json");
    xhr.responseType = "blob"; // 返回类型blob   
    //定义请求完成的处理函数，请求前也可以增加加载框/禁用下载按钮逻辑
    xhr.onload = function() {
    // 请求完成
      if (this.status === 200) {
        // 返回200
        var blob = this.response;
        var url = window.URL.createObjectURL(blob);
        var fileName = policyNo + ".pdf";
        var link = document.createElement("a");
        link.style.display = "none";
        link.href = url;
        link.setAttribute("download", fileName);
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
      }
    }; // 发送ajax请求
    xhr.send(data);
  ```
* blob下载文件  <https://www.cnblogs.com/goloving/p/7651636.html> 
* 原生ajax请求  <https://www.jianshu.com/p/2be2e4f1fc8e>

***

   
   

### 2 jquery封装ajax使用formData进行文件上传
***
##### 2-1 问题描述
通过android方法调用相机，H5接收android返回的图片base64码传给后端，后端没有反应
##### 2-2 问题原因-解决方法
 processData 这玩意儿在使用 FormData 作为数据体时是不能开启的而 $.post 里默认又是开启的所以只能用 $.ajax 这个方法
 ```
      $.ajax({
          url: "/zyvp/media/imageUpload",
          type: "POST",
          data: param,
          contentType: false, // 关关关！必须得 false
          // 这个不关会扔一个默认值application/x-www-form-urlencoded过去，后端拿不到数据的！
          // 而且你甚至不能传个字符串'multipart/form-data'，后端一样拿不到数据！
          processData: false,
          success: function(response) {
            if (response.status === 1) {
              var data = JSON.stringify({
                orderNo: policyNo,
                operateType: "ocrRecognise",
                recogniseType: "recognize_id_card",
                fileName: response.data.fileName,
                authUserName: "zyic",
                authUserPwd: "zyic"
              });
              $.ajax({
                type: "POST",
                url: "/zyvp/media/ocrRecognise",
                contentType: "application/json;charset=utf-8",
                data: data,
                success: function(response) {
                  if (response.status == 1) {
                    $("#idNumber").val(response.data.id_number);
                  } else {
                  }
                },
                error: function(error) {
                  alert(error);
                }
              });
            } else if (response.data.status === 0) {
              alert("上传失败!");
            }
          },
          error: function(error) {}
        });
 ```
 ##### 2-3 关于formData
 * 使用FormData对象来传输文件，FormData 对象的字段类型可以是  **Blob, File, string** 如果它的<font color='red'>**字段类型不是Blob也不是File，则会被转换成字符串类型**</font>。

 * 使用append添加时formdata的key若已存在，则不能重复添加，会忽略本次append操作，所以建议set替代append来使用
 
###### 参考文档    <http://www.bubuko.com/infodetail-2927785.html>
*** 