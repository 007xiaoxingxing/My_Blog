---
title: TPLINK系列路由器新版UI登录算法
categories:
  - Program
tags:
  - router
date: 2017-06-02 16:18:32
---

因为某些原因需要用程序来向路由器提交宽带帐号和密码，进行拨号等操作。原本的TP系的路由器使用的是Basic认证，只要在http请求头中加上个Authtication头就可以通过认证了，某些型号也最多要求个Referer或者cookie，但是新版的UI登录的过程需要在浏览器端把密码加密后再发送到路由以获得一个token来通过认证。下面记录一下我的分析过程。

<!-- more -->

经过寻找，加密算法存在与jquery.js这个文件中（一直以为只是引用了jquery库，谁曾想关键代码都在这里边）。

```javascript
 securityEncode: function(a, b, c) {
            var e = "",
            f, g, h, k, l = 187,
            n = 187;
            g = a.length;
            h = b.length;
            k = c.length;
            f = g > h ? g: h;
            for (var p = 0; p < f; p++) n = l = 187,
            p >= g ? n = b.charCodeAt(p) : p >= h ? l = a.charCodeAt(p) : (l = a.charCodeAt(p), n = b.charCodeAt(p)),
            e += c.charAt((l ^ n) % k);
            return e
        },
        orgAuthPwd: function(a) {
            return $.securityEncode("RDpbLfCPsJZ7fiv", a, "yLwVl0zKqws7LgKPRQ84Mdt708T1qQ3Ha7xv3H7NyU84p21BriUWBU43odz3iP4rBL3cD02KZciXTysVXiV8ngg6vL48rPJyAUw0HurW20xqxv9aYb4M9wK1Ae0wlro510qXeU07kV57fQMc8L6aLgMLwygtc0F10a0Dg70TOoouyFhdysuRMO51yY5ZlOZZLEal1h0t9YQW0Ko7oBwmCAHoic4HYbUyVeU3sfQ1xtXcPcf1aT303wAQhv66qzW")
        },
```

关键的加密过程的算法都在这里了。

然后用C#重写一下：

```c#
public  string securityEncode(string a, string b, string c)
        {
            string e = "";
            int f, g, h, k, l = 187, n = 187;
            g = a.Length;//短验证码的长度(固定为15)  
            h = b.Length;//密码长度  
            k = c.Length;//长验证码的长度(固定为255)  
            if (g > h)//将短验证码字符串和密码字符串的长度进行比较  
            {
                f = g;
            }
            else
            {
                f = h;
            }
            //f取长的那个字符串的长度  
            for (int p = 0; p < f; p++)
            {
                n = l = 187;
                if (p >= g)
                {
                    n = b.Substring(p, 1).ToCharArray()[0];//n取密码中的以0开始的第p位字符  
                }
                else
                {
                    if (p >= h)
                    {
                        l = a.Substring(p, 1).ToCharArray()[0];//l取短验证码中的以0开始的第p位字符  
                    }
                    else
                    {
                        l = a.Substring(p, 1).ToCharArray()[0];//l取短验证码中的以0开始的第p位字符  
                        n = b.Substring(p, 1).ToCharArray()[0];//n取密码中的以0开始的第p位字符  
                    }
                }
                //每次计算出l和n的值之后对其取异或然后除以k(也就是除以255),  
                //取长验证码中以0开始的第(l^n)%k位，然后拼接到字符串e的后面  
                e += c.Substring((l ^ n) % k, 1);
            }

            return e;//返回加密后的密码  
        }
```

利用C#模拟登录路由器：

```c#
 public string Login886(string data,string url) {

            string post_data = "{\"method\":\"do\",\"login\":{\"password\":\""+data+"\"}}";
            byte[] encode_data = System.Text.Encoding.UTF8.GetBytes(post_data);
            

            HttpWebRequest request = (HttpWebRequest)WebRequest.Create(url);
            request.Method = "POST";
            request.ContentLength = encode_data.Length;
            request.Headers.Add("Origin", url);
            request.UserAgent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.96 Safari/537.36";
            request.ContentType = "application/json; charset=UTF-8";
            request.Accept = "application/json, text/javascript, */*; q=0.01";
            request.Headers.Add("X-Requested-With", "XMLHttpRequest");
            request.Referer = url;
            request.Headers.Add("Accept-Encoding", "gzip, deflate");
            request.Headers.Add("Accept-Language", "zh-CN, zh; q=0.8,en; q=0.6,ja; q=0.4");

            //这句代码一定要加上，不然路由器不会响应登录请求
            request.ServicePoint.Expect100Continue = false;
            
            Stream myRequestStream = request.GetRequestStream();
           
            myRequestStream.Write(encode_data, 0, encode_data.Length);
            myRequestStream.Close();

            HttpWebResponse response = (HttpWebResponse)request.GetResponse();

            Stream myResponseStream = response.GetResponseStream();
            StreamReader myStreamReader = new StreamReader(myResponseStream, Encoding.GetEncoding("utf-8"));
            string retString = myStreamReader.ReadToEnd();
            myStreamReader.Close();
            myResponseStream.Close();

            return retString;
           

        }
```

完



参考资料： http://blog.csdn.net/dev_here/article/details/51235324