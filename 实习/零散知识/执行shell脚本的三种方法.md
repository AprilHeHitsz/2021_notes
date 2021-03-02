2020/01/20
 sh：当前shell是父进程，生成一个shell子进程，在shell中执行脚本；
- source：在当前上下文中执行脚本，不会生成新进程。脚本执行完毕，回到当前shell；
- exec：用command进程替代当前进程，保持pid不变，执行完毕直接退出，不回到当前shell环境