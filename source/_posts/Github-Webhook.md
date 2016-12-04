---
title: 利用Github的webhook和tornado实现代码自动部署
date: 2016-12-04 10:39:31
categories: Program
tags:
- Github
- Webhook
- 自动部署
- code deploy
---

### 0x1:需求来源  

本博客使用的hexo生成的静态页面，整个博客代码是托管在github上的。这样可以方便我在不同的设备上随时同步位于github上的博客代码，并撰写新的内容。写博文的问题得到了解决，但是每次更新服务器上的内容时却要登录服务器去拉取github上的内容，很是繁琐。github提供了webhook功能，用户对仓库操作引起的事件，例如push，pull等可以触发github利用webhook向远程服务器发起链接，并携带一定的参数，这样的话，在远程服务器监听来自webhook的请求就可以执行代码更新操作了哇。  

### 0x2:服务端代码

服务端代码的思路很清晰。 

1. 监听来自webhook的消息，并验证是否合法  
2. 拉取git仓库的内容至本地临时仓库
3. 拷贝代码至web根目录，并删除web部署目录中的.git目录，防止代码泄露  

正好最近在学习python，就利用python的tornado框架写几句话完成服务端的工作吧。

```python
import tornado.ioloop
import tornado.web
import json
import sys
import os

#pull my blog from github
def PullBlog():
	os.system("cd ~/blog/My_Blog&&git pull https://github.com/007xiaoxingxing/My_Blog.git&&cp -r public/* /var/blog/")
	print "pull blog"

class MainHandler(tornado.web.RequestHandler):
	def get(self):
		self.write("Hello Guys!");
	def post(self):
		json_str=self.request.body
		data=json.loads(json_str)
        #这里的内容需要根据github的post数据自行进行判断，我这里一切从简了
		target= data["repository"]["name"]
		if target == "My_Blog":
			PullBlog()
			print "ooo"
		if target == "photo_manage":
			PullPhoto()
application = tornado.web.Application([
	(r"/",MainHandler),
])

if __name__ == "__main__":
	application.listen(8888)
	tornado.ioloop.IOLoop.instance().start()

```

这段代码是不是很眼熟？Yes，它就是tornado的Hello World，我在其中增添了获取post内容，并把接收到的json字符串解析成python对象，如果符合要求就进行代码更新操作。

### 0x3:Github代码仓库相关设置

在github代码仓库需要设置webhook的远程服务器地址，post的数据格式和密钥。

![image](/blogimg/githook.png)

设置好webhook的url，并点击Add webhook按钮后，github的服务器会第一次发送数据到你设置的url地址上，我们来看一下它都发送了些什么数据。

```json
{
  "ref": "refs/heads/master",
  "before": "2db041797c019dd3f04b45203ba4ad09ffef2654",
  "after": "454cc4e6bfce259cd29fbc112c9b3714e0750ab7",
  "created": false,
  "deleted": false,
  "forced": false,
  "base_ref": null,
  "compare": "https://github.com/007xiaoxingxing/My_Blog/compare/2db041797c01...454cc4e6bfce",
  "commits": [
    {
      "id": "454cc4e6bfce259cd29fbc112c9b3714e0750ab7",
      "tree_id": "09538c4e1f362018d9e230d4de2225ca469fb32a",
      "distinct": true,
      "message": "PIL grab",
      "timestamp": "2016-12-04T10:38:02+08:00",
      "url": "https://github.com/007xiaoxingxing/My_Blog/commit/454cc4e6bfce259cd29fbc112c9b3714e0750ab7",
      "author": {
        "name": "007xiaoxing",
        "email": "chanxing9@gmail.com",
        "username": "007xiaoxingxing"
      },
      "committer": {
        "name": "007xiaoxing",
        "email": "chanxing9@gmail.com",
        "username": "007xiaoxingxing"
      },
      "added": [
        "public/2016/12/04/PIL-Grab/index.html",
        "public/archives/2016/12/index.html",
        "public/tags/PIL/index.html",
        "public/tags/python/index.html",
        "source/_posts/PIL-Grab.md"
      ],
      "removed": [

      ],
      "modified": [
        "db.json",
        "public/2016/07/22/Learn-Markdown/index.html",
        "public/2016/07/30/初识OD/index.html",
        "public/2016/11/07/hello-world/index.html",
        "public/2016/11/20/GDB-peda/index.html",
        "public/2016/11/27/Pwnable-fd/index.html",
        "public/2016/11/28/OpenWRT-BuildEnvironMent/index.html",
        "public/2016/11/28/Pwnable-collision/index.html",
        "public/2016/11/29/C-Helloworld/index.html",
        "public/2016/11/29/HCTF2016-Writeup/index.html",
        "public/2016/11/29/Learn_VIM/index.html",
        "public/2016/11/30/Github-Add-SSHKEY/index.html",
        "public/about/index.html",
        "public/archives/2016/07/index.html",
        "public/archives/2016/11/index.html",
        "public/archives/2016/index.html",
        "public/archives/2016/page/2/index.html",
        "public/archives/index.html",
        "public/archives/page/2/index.html",
        "public/categories/CTF/index.html",
        "public/categories/Linux/index.html",
        "public/categories/Program/index.html",
        "public/index.html",
        "public/page/2/index.html",
        "public/tags/C/index.html",
        "public/tags/CTF/index.html",
        "public/tags/Debugger/index.html",
        "public/tags/Edtor/index.html",
        "public/tags/GDB/index.html",
        "public/tags/HCTF/index.html",
        "public/tags/Helloworld/index.html",
        "public/tags/Linux/index.html",
        "public/tags/PWN/index.html",
        "public/tags/VIM/index.html",
        "public/tags/enviroment/index.html",
        "public/tags/github/index.html",
        "public/tags/markdown/index.html",
        "public/tags/openwrt/index.html",
        "public/tags/peda/index.html",
        "public/tags/pwnable/index.html",
        "public/tags/ssh-key/index.html",
        "public/tags/tools-reverse/index.html",
        "public/tags/入坑/index.html",
        "public/tags/萌新第一次/index.html"
      ]
    }
  ],
  "head_commit": {
    "id": "454cc4e6bfce259cd29fbc112c9b3714e0750ab7",
    "tree_id": "09538c4e1f362018d9e230d4de2225ca469fb32a",
    "distinct": true,
    "message": "PIL grab",
    "timestamp": "2016-12-04T10:38:02+08:00",
    "url": "https://github.com/007xiaoxingxing/My_Blog/commit/454cc4e6bfce259cd29fbc112c9b3714e0750ab7",
    "author": {
      "name": "007xiaoxing",
      "email": "chanxing9@gmail.com",
      "username": "007xiaoxingxing"
    },
    "committer": {
      "name": "007xiaoxing",
      "email": "chanxing9@gmail.com",
      "username": "007xiaoxingxing"
    },
    "added": [
      "public/2016/12/04/PIL-Grab/index.html",
      "public/archives/2016/12/index.html",
      "public/tags/PIL/index.html",
      "public/tags/python/index.html",
      "source/_posts/PIL-Grab.md"
    ],
    "removed": [

    ],
    "modified": [
      "db.json",
      "public/2016/07/22/Learn-Markdown/index.html",
      "public/2016/07/30/初识OD/index.html",
      "public/2016/11/07/hello-world/index.html",
      "public/2016/11/20/GDB-peda/index.html",
      "public/2016/11/27/Pwnable-fd/index.html",
      "public/2016/11/28/OpenWRT-BuildEnvironMent/index.html",
      "public/2016/11/28/Pwnable-collision/index.html",
      "public/2016/11/29/C-Helloworld/index.html",
      "public/2016/11/29/HCTF2016-Writeup/index.html",
      "public/2016/11/29/Learn_VIM/index.html",
      "public/2016/11/30/Github-Add-SSHKEY/index.html",
      "public/about/index.html",
      "public/archives/2016/07/index.html",
      "public/archives/2016/11/index.html",
      "public/archives/2016/index.html",
      "public/archives/2016/page/2/index.html",
      "public/archives/index.html",
      "public/archives/page/2/index.html",
      "public/categories/CTF/index.html",
      "public/categories/Linux/index.html",
      "public/categories/Program/index.html",
      "public/index.html",
      "public/page/2/index.html",
      "public/tags/C/index.html",
      "public/tags/CTF/index.html",
      "public/tags/Debugger/index.html",
      "public/tags/Edtor/index.html",
      "public/tags/GDB/index.html",
      "public/tags/HCTF/index.html",
      "public/tags/Helloworld/index.html",
      "public/tags/Linux/index.html",
      "public/tags/PWN/index.html",
      "public/tags/VIM/index.html",
      "public/tags/enviroment/index.html",
      "public/tags/github/index.html",
      "public/tags/markdown/index.html",
      "public/tags/openwrt/index.html",
      "public/tags/peda/index.html",
      "public/tags/pwnable/index.html",
      "public/tags/ssh-key/index.html",
      "public/tags/tools-reverse/index.html",
      "public/tags/入坑/index.html",
      "public/tags/萌新第一次/index.html"
    ]
  },
  "repository": {
    "id": 64537807,
    "name": "My_Blog",
    "full_name": "007xiaoxingxing/My_Blog",
    "owner": {
      "name": "007xiaoxingxing",
      "email": "994983825@qq.com"
    },
    "private": false,
    "html_url": "https://github.com/007xiaoxingxing/My_Blog",
    "description": null,
    "fork": false,
    "url": "https://github.com/007xiaoxingxing/My_Blog",
    "forks_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/forks",
    "keys_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/keys{/key_id}",
    "collaborators_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/collaborators{/collaborator}",
    "teams_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/teams",
    "hooks_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/hooks",
    "issue_events_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/issues/events{/number}",
    "events_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/events",
    "assignees_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/assignees{/user}",
    "branches_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/branches{/branch}",
    "tags_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/tags",
    "blobs_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/git/blobs{/sha}",
    "git_tags_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/git/tags{/sha}",
    "git_refs_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/git/refs{/sha}",
    "trees_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/git/trees{/sha}",
    "statuses_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/statuses/{sha}",
    "languages_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/languages",
    "stargazers_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/stargazers",
    "contributors_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/contributors",
    "subscribers_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/subscribers",
    "subscription_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/subscription",
    "commits_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/commits{/sha}",
    "git_commits_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/git/commits{/sha}",
    "comments_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/comments{/number}",
    "issue_comment_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/issues/comments{/number}",
    "contents_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/contents/{+path}",
    "compare_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/compare/{base}...{head}",
    "merges_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/merges",
    "archive_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/{archive_format}{/ref}",
    "downloads_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/downloads",
    "issues_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/issues{/number}",
    "pulls_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/pulls{/number}",
    "milestones_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/milestones{/number}",
    "notifications_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/notifications{?since,all,participating}",
    "labels_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/labels{/name}",
    "releases_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/releases{/id}",
    "deployments_url": "https://api.github.com/repos/007xiaoxingxing/My_Blog/deployments",
    "created_at": 1469872695,
    "updated_at": "2016-11-29T12:03:47Z",
    "pushed_at": 1480819114,
    "git_url": "git://github.com/007xiaoxingxing/My_Blog.git",
    "ssh_url": "git@github.com:007xiaoxingxing/My_Blog.git",
    "clone_url": "https://github.com/007xiaoxingxing/My_Blog.git",
    "svn_url": "https://github.com/007xiaoxingxing/My_Blog",
    "homepage": null,
    "size": 16987,
    "stargazers_count": 0,
    "watchers_count": 0,
    "language": "HTML",
    "has_issues": true,
    "has_downloads": true,
    "has_wiki": true,
    "has_pages": false,
    "forks_count": 0,
    "mirror_url": null,
    "open_issues_count": 0,
    "forks": 0,
    "open_issues": 0,
    "watchers": 0,
    "default_branch": "master",
    "stargazers": 0,
    "master_branch": "master"
  },
  "pusher": {
    "name": "007xiaoxingxing",
    "email": "994983825@qq.com"
  },
  "sender": {
    "login": "007xiaoxingxing",
    "id": 8982713,
    "avatar_url": "https://avatars.githubusercontent.com/u/8982713?v=3",
    "gravatar_id": "",
    "url": "https://api.github.com/users/007xiaoxingxing",
    "html_url": "https://github.com/007xiaoxingxing",
    "followers_url": "https://api.github.com/users/007xiaoxingxing/followers",
    "following_url": "https://api.github.com/users/007xiaoxingxing/following{/other_user}",
    "gists_url": "https://api.github.com/users/007xiaoxingxing/gists{/gist_id}",
    "starred_url": "https://api.github.com/users/007xiaoxingxing/starred{/owner}{/repo}",
    "subscriptions_url": "https://api.github.com/users/007xiaoxingxing/subscriptions",
    "organizations_url": "https://api.github.com/users/007xiaoxingxing/orgs",
    "repos_url": "https://api.github.com/users/007xiaoxingxing/repos",
    "events_url": "https://api.github.com/users/007xiaoxingxing/events{/privacy}",
    "received_events_url": "https://api.github.com/users/007xiaoxingxing/received_events",
    "type": "User",
    "site_admin": false
  }
}
```

数据内容十分丰富，包括了你对仓库的操作，仓库改变等等......  

这些数据都可以在服务端进行获取，验证等，然后自定义操作即可。

### 0x4:服务的运行

最后只需要将自动部署的服务端跑起来即可。我选择使用nohub将服务挂起来在后台运行。

```bash
$nobub python git_deploy.py &
```

nohub会把后台运行的标准输出重定向到当前目录的nohub.dat中，在这之中我们可以查看相关操作日志。Ok，大功告成，以后就可以直接提交代码的github代码仓库，服务器就能够自动的更新博客文章了。