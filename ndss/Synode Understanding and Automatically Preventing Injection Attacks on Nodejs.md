source: https://www.teaspect.com/detail/5764

Node.js生态系统已经创造了许多的现代应用程序，其中包括服务器侧的Web应用程序和桌面应用程序。与浏览器中的JavaScript应用程序不同的是，Node.js应用程序没有浏览器沙箱的保护，可以和操作系统自由交互。因此，针对Node.js的注入攻击可以带来重大的损害。本文对235850个 Node.js模块进行了大规模研究，发现数千个模块可能遭受注入攻击，并提出了Synode，一种自动化的注入缓解技术。Synode结合了静态分析和动态的运行时安全策略，易于部署，不需要对Node.js平台进行任何修改，其运行开销也达到了亚毫秒级，并且误报率只有10%。

一、JavaScript的应用场景革新和问题

JavaScript除了驱动了绝大多数运行在浏览器中的Web应用程序，还在其他的平台有着革命性的进步。服务器侧Web应用程序和桌面应用程序都在越来越广泛地采用Node.js；移动应用程序也在越来越多地采用JavaScript，甚至操作系统，例如Firefox OS，也开始采用JavaScript进行开发。

JavaScript应用场景的革新，也将传统应用场景中JavaScript的问题传播到了其他的场景。值得注意的是这种问题是JavaScript语言本身在开发中可能导致的脆弱点，而不是简单的将Web应用程序的问题传播到其他场景。比如注入漏洞，在移动应用程序和Node.js环境中的影响更加严重，而浏览器Web应用中的最大问题则是XSS。

二、Synode整体框架

Synode将静态分析和运行时检测结合到一个全自动的方法中，以缓解注入攻击，这是第一个解决Node.js中注入攻击的方法。Synode的总体思想是防止在注入API的注入。Synode首先会分析可能流入注入API的值，如果没有不受信任的数据可能流入注入API，则不需要进行进一步的分析；反之，如果有不受信任的数据可能流入注入点，则需要进行动态的运行时检测。对于运行时检测，Synode的方法是在静态分析时重写源代码，将检查代码插入到注入点之前，当运行时检测器检测到违反安全策略的值，程序将会终止，以防止注入攻击。


三、静态分析

Synode的静态分析器首先将传递给注入API的值解析为树；然后对树进行分析得到一组模板，这些模板表示传递给函数的可能字符串的值，包括已知和未知部分；最后，基于生成的模板，静态分析器分析注入API的每个调用点是否是静态安全的，从而进一步决定是否要插入运行时检测代码。


四、动态运行时检测

对于无法确定传递给注入API的值的调用点，采用动态的运行时检测的机制。运行时检测器分两步实现预期目标。

首先，检测器将静态提取的调用点模板集，转换为表示良性值的一组部分语法树（PAST），而模板的未知部分，则表示为未知子树；在执行时，检测器将传递给注入API的值解析为抽象语法树（AST），扩展未知子树。在这一步骤中，检测器将强制执行一种策略，使这个AST必须能够从至少一个部分语法树中导出，并且这些扩展要在所有可能AST节点的允许子集范围内。

五、性能

作者使用了搭载Intel Core I7(2.10GHz)，12GB 内存，运行Ubuntu 14.04的联想ThinkPad T440s笔记本进行测试。

对于静态分析，Synode成功完成了15604个模块中的96.27%，平均分析时间是4.38秒；对于运行时检测，每一次检测的成本为0.74毫秒，在实践中，这一开销通常可以忽略不计。
