# D-WASM

D-WASM 是 Decompile-WASM 的简称，这里将列出现有逆向WASM程序的相关资料以及痛点分析；嗷，非常建议先阅读 JEB 的逆向入门文档 <a href="https://www.pnfsoftware.com/reversing-wasm.pdf">Reversing WebAssembly</a> 其痛点的总结也比下文更到位，并有一部分 WASM 的基础入门，同时 <a href="https://www.forcepoint.com/blog/x-labs/analyzing-webassembly-binaries">Analyzing WebAssembly binaries </a>也是很不错的教程。

开门见山的说，WASM逆向的工具越来越多了并且也会越来越成熟，这里做个记录供需要者采用。





### Significance

最早接触 WASM 是在了解<a href="https://bellard.org/jslinux/">线上虚拟机</a>的时候，还有 asm.js 等，然后就了解到了 web assembly。有关的知识细节可以参考官网，网络中也有较多的资料。总而言之，接触过 WASM 的开发人员都清楚 Ending 定律：“一切能编译成 WASM 的，终将会被编译成 WASM”。从这个角度看，针对 WASM 程序的安全分析也会愈来愈重要。

当然，WASM 程序设计的初衷就是安全的——沙箱化的运行方式使得其少有C语言类别的漏洞，语言本身的安全性如果能保证的话，分析程序的安全性则成为重点，而二进制分析的重点，自然就是如何逆向了。



### Challenge

逆向 WASM 程序具有许多困难，在这里做简单的总结

1. **语言接口较多**：现在 WASM 程序已经可以由多款高级语言编译得到，如<i>C++, C, C#, Python, GO, Rust </i>等。由于 WASM 本身良好的迁移性，使得其成为这些上层语言青睐的对象。而这也使得 WASM 的逆向分析充满了挑战，试想拿到 WASM 代码后，若不清楚是由哪一款语言编译得到的，自然会多了顾虑；
2. **动态分析工具的缺乏**：即使已经有了几款 WASM 引擎，但 WASM 相关的动态调试工具是缺乏的；实际上，模拟 WASM 的栈执行并不复杂，数百行代码也可以自制一个虚拟机，如 <a href="https://github.com/EVulHunter/EVulHunter/blob/master/myhelper/wasmvm.py">EVulHunter</a> 下的代码就可以作为学习的样例；更复杂的有如 <a href="https://github.com/wasmerio/wasmer">wasmer </a>等。但对于完全的动态调试，是不够的，笔者认为原因如下
   1. 外部函数 / 胶水代码实现：WASM 代码有许多 import 的模块，如果想要完全动态调试，则应该也完成这些代码；
   2. 现除了运行在浏览器引擎上外，多数自制的虚拟机也问世。如区块链EOS就部署了自己的WASM VM，其代码内嵌于链代码中，即使是像其他代码可以在浏览器调试中设置断点那样也是不行的。



### Known Tools and Methods

- 浏览器动态调试：

  这也是在比赛中见的比较多的情况，此时的 WASM 代码基本也只是扮演了 “复杂的JS” 的角色，由于有像 Chrome 和 Firefox 这样的开发者工具，使得单步、断点跟踪 WASM 程序是可能的。在程序执行的平台上逆向是最轻松的，胶水代码都准备完成，浏览器也帮助你划分了模块的各个函数。

  即使有调试的支持，浏览器逆向的工作仍是非常繁琐的——无法注释，工作易间断；函数名等支持较弱，难以定位到关键函数；还有，每次查看内存的流程都太太太麻烦了（读者可以亲自试试要点开几层按钮）这些缺点在逆向大型程序的时候往往有致命的效果；

  即使如此，这个方法也是目前最有效的动态调试策略

  

- <a href="https://www.radare.org/">Radare2</a>
  R2 自然是程序分析领域的神器了，搭配 cutter 或者官方的 GUI 使得其使用起来也额外方便。同时和 r2pipe 一起使用可以节约大量人工工作

  但奇怪的是，我在 10.14.5 MAC 版本下的 radare2 + cutter 并没有正确的反汇编 wasm，所以这个方式我只是简单记录一下
  
  

- <a href="https://github.com/WebAssembly/wabt">wabt</a>

  把 wabt (WebAssembly binary toolkit) 作为逆向 wasm 的最专业程序是不过分的，由于其本身就是 wasm 设计者那批人所研究的，并有比较活跃的更新。如果要说逆向 WASM 汇编有个标准的话，我认为工具内的 `wasm2wat` 一定是基准；当然工具箱内还有其他的工具如`wasm2c`，即使其输出的C代码很多时候还不如看汇编（纯属个人意见）

  缺点就是相比一些其他的 python 脚本而言拓展性不足。其宣称 "easier integration into other systems"对于非专业 C/C++ 编程人士而言还是太刺耳了



- <a href="https://github.com/fireeye/idawasm">Idawasm</a>

  提到静态分析就会让人想到 IDA，也有大牛写了关于 WASM 的插件（不过我至今没有成功加载）如果版本对的上的话或许可以尝试一下
  
  

- <a href="https://github.com/wwwg/wasmdec">wasmdec</a>

  编写 wasmdec 的人一定也是一个 ctf 爱好者，或许其可以用于解决一些简单的程序，但是却已经很久没有更新了，提供的online接口也会有比较难以 handle 的小错误；相比之下，或许 wabt 会更可靠一些

  

- <a href="https://github.com/quoscient/octopus">Octopus</a>

  octopus是一个相当方便的工具了，首创的提供了 cfg 的模块，并且易于拓展的 python 模块也是很受人青睐。查看其主页已经有 20 + 的项目是基于它的了，但可惜的是其本身的获赞数并不高😂，octupos的内核是 <a href="https://pypi.org/project/wasm/">wasm 1.2</a>，用户可以基于此编写自己的应用



- <a href="http://wasabi.software-lab.org/">wasabi</a>

  octopus的作者确实非等闲之辈，其第二套wasm工具已经有了动态工具的雏型——通过插入桩代码来增强已有的动态分析，这是一个非常新颖的点子。程序是基于 rust 的，支持用户自己编写 `analysis.js`，虽然还不太清楚它是如何解决胶水代码的相关问题的，但这个工具的创新或推动逆向继续前进



- <a href="https://www.pnfsoftware.com/jeb/">JEB</a>

  如果说有一个用户友好的静态分析工具可以适合使用的话，那就是开头文章附录里提到的 JEB decompiler 了～你可以下载文档查看如何使用，方便的 GUI 会是你选择它的一个重要的理由



（如果上文中遗漏了什么重要的工具的话，请联系我；）

### Conclusions

WASM 程序问世已经超过2年了，其取得成就或许没有当初期望的那么大，但也是不容忽视的。而有关它的一整套的分析美学要什么时候有体系呢？