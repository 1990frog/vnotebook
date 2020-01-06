[TOC]

这实际上是三个问题，从WebService到今天流行的RESTful API(JSON) over HTTP，经历了数次变革

# WebService有很多协议，为什么HTTP比较流行？
WebService是个很重型的规范，它的应用协议是SOAP（简单对象访问协议），它所依赖的下层通信方式不单单是HTTP，也有SOAP over SMTP, SOAP over TCP，由于HTTP协议群众基础广，开发调试方便，所以，成了WebService中最为流行的方式。

甚至很多公司在内网通信，也用HTTP来做，比如，应用调用搜索引擎，Solr就是一个例子。

但HTTP也是TCP上性能比较差的协议，因为HTTP是基于TCP的，有3次握手，再加上HTTP是个文本传输协议（虽然也可以传二进制的附件， 但业务逻辑还是文本用的多），又有很多复杂的HEADER。所以人们发明了一些更高效的通信协议来做远程调用，比如ACE、ICE、Corba、淘宝的 HSF，但这是后话了，不展开细说。你只要知道，HTTP之所以流行，乃是简单易用群众基础广的结果。

# WebService为什么不如RESTful API流行
WebService诞生十几年了，最初是IBM、微软比较热心在推，一直也不温不火。倒是XML-RPC, RESTful以及比RESTful还要简陋的远程调用方式后来居上。感觉是不是有点像民间的Spring干掉官方的EJB？

究其原因，还是WebService实在太笨重了，SOAP信封犹如婆娘的裹脚布，又臭又长，广大开发人员是叔可忍嫂不能忍，于是就有了简化版的， 叫XML-RPC，后来伴随着Web2.0流行，RESTful独领风骚。我在10年前做过一个产品，纯PHP+JS，标准的WebService，连 WSDL我都要专门写个PHP程序来生成，还好只是我一个人开发，要是团队协作，我早就被骂得不成人形了。

再后来，连RESTful都被嫌弃了，大伙儿干脆连PUT、DELETE都懒得用，直接用GET和POST。

同时，我得说，这只是在互联网领域，大部分企业的业务逻辑相对简单，同时工期又变态的短（就像大部分互联网创业公司用糙快猛的PHP，而不用相对严 谨的Java一样）。在某些业务复杂，稳定性和正确性要求高的领域（如ERP、电商、支付），WebService还有是用武之地的。

# 为什么JSON比XML流行
还是易用性，JSON的可读性比XML强几条长安街，解析规则也简单许多。XML解析的时候规则太多了，动不动就非法字符，动不动就抛异常。这对追求高开发速度和低开发门槛的企业来说，是个致命伤。

JSON的缺点是数据类型支持较少，且不精确。比方说：
```
price:12580
```
在json里，你无法知道这个价格是int, float还是double。

所以，如上面第二条所述，在一些业务要求较高的领域，还是XML更合适。

最后说一下性能，JSON的性能高于XML，除此之外，基于XML和HTTP的WebService, 基于JSON的RESTful API，并没有性能差异。

XML性能糟糕到什么地步呢，有一种专门的CPU叫做XML Accelerator，专门为XML解析提供硬件加速。

