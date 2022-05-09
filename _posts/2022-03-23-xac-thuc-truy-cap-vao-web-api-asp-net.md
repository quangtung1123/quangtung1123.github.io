---
layout: post
title:  "Xác thực truy cập vào web API ASP.NET"
date:   2022-03-13 10:10:00
permalink: 2022/03/23/xac-thuc-truy-cap-vao-web-api-asp-net
tags: Linux
category: Linux
img: /assets/xac-thuc-truy-cap-vao-web-api-asp-net/Hinh1.png
summary: Xác thực truy cập vào web API ASP.NET

---

1) Tạo một dự án ứng dụng web asp.net mới.

2) Chọn Ứng dụng web asp.net Empty và tick chọn MVC và Web API.

3) Tạo một thư mục 'MessageAPIHandler' và thêm tệp lớp là 'AuthorizationHandler.cs'.

```C#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net.Http;
using System.Threading;
using System.Threading.Tasks;
using System.Web;

namespace WebAPI.MessageAPIHandler
{
    public class AuthorizationHandler : DelegatingHandler
    {
        public const string APIKeyConfigured = "admin";

        //public string APIKeyConfigured = System.Configuration.ConfigurationManager.AppSettings["APIKeyConfigured"].ToString();

        protected override async Task<HttpResponseMessage> SendAsync(HttpRequestMessage request, CancellationToken cancellationToken)
        {
            IEnumerable<string> headerAPIkeyValues = null;
            if (request.Headers.TryGetValues("X-ApiKey", out headerAPIkeyValues))
            {
                var secretKey = headerAPIkeyValues.First();

                if (!string.IsNullOrEmpty(secretKey) && APIKeyConfigured.Equals(secretKey))
                {
                    return await base.SendAsync(request, cancellationToken);
                }
            }
            return request.CreateResponse(System.Net.HttpStatusCode.Forbidden, "API key is invalid.");
        }
    }
}
```

4) Thêm mã bên dưới trong Application_Start Event Handler của Global.asax.cs

```C#
GlobalConfiguration.Configuration.MessageHandlers.Add(new AuthorizationHandler());
```

Thuộc tính [FromBody] được sử dụng để lấy giá trị của mục Nhân viên từ phần thân của yêu cầu HTTP.
HTTP GETmethod được chỉ định bởi thuộc tính [HttpGet] và phương thức HTTP POST được chỉ ra bởi thuộc tính [HttpPost]

**Tài liệu tham khảo**

- [codingpointer](https://www.codingpointer.com/asp-dot-net-tutorial/rest-web-api-key-authentication)

---
