# lab0 SPOC思考题

## 个人思考题

---

能否读懂ucore中的AT&T格式的X86-32汇编语言？请列出你不理解的汇编语言。
- [x]  

>  http://www.imada.sdu.dk/Courses/DM18/Litteratur/IntelnATT.htm
能基本读懂汇编语言。

虽然学过计算机原理和x86汇编（根据THU-CS的课程设置），但对ucore中涉及的哪些硬件设计或功能细节不够了解？
- [x]  

>   外部设备访问，中断，x86汇编的各种运行模式。


哪些困难（请分优先级）会阻碍你自主完成lab实验？
- [x]  

>   对实验整体的把握，不懂该干什么（因为以往每次实验都是刚开始不知道如何做，后来慢慢上手之后就稍微轻松点），所以希望前面可以做些演示
课堂内容理解不足，造成实验困难

如何把一个在gdb中或执行过程中出现的物理/线性地址与你写的代码源码位置对应起来？
- [x]  

>  即虚拟地址（用户使用）与物理地址的映射关系，建立虚表，利用虚拟地址计算出物理地址。

了解函数调用栈对lab实验有何帮助？
- [x]  

>   函数调用时，调用者依次把参数压栈，然后调用函数，函数调用以后，在堆栈中取得数据，并进行计算。函数计算结束以后，或者调用者，或者函数本身修改堆栈，使得堆栈恢复原装。如果了解这个过程的话，就清晰程序的整个运行过程，那这样在实现中断或者其他一些实验功能的时候就会更方便一些。

你希望从lab中学到什么知识？
- [x]  

>   了解操作系统如何组织协调各个硬件配合工作，了解操作系统能提供给上层应用的接口有哪些，上层应用该怎么使用等，总之就是更全面透彻了解软件在计算机上运行的一个原理。

---

## 小组讨论题

---

搭建好实验环境，请描述碰到的困难和解决的过程。
- [x]  

> 导入vdi文件 
  解决过程：咨询百度

熟悉基本的git命令行操作命令，从github上
的 http://www.github.com/chyyuu/ucore_lab 下载
ucore lab实验
- [x]  

> A. 新建Git仓库，创建新文件夹    git init
  B. 添加文件到git索引    git add <filename>  --- 单个文件添加
                          git add *　　--- 全部文件添加
  C. 提交到本地仓库       git commit -m "代码提交描述"
  D. 提交到远端仓库       git push origin master      ***master可以换成你想要推送的任何分支
  分支：
    1. 创建一个叫做"lee"的分支，并切换过去      git checkout -b lee
    2. 切换回主分支       git checkout master
    3. 把新建的分支删除     git branch -d lee
    4. 再push分支到远端仓库前，该分支不被人所见到 git push origin <branch>
  更新与合并
    A. 更新本地仓库 git pull
    B. 自动合并分支，多时引起冲突，冲突后需要手动解决 git merge <branch>
    C. 合并后需要添加 git add <branch>
    D. 合并前建议使用对比工具 git diff <source_branch> <target_branch>
    E. 软件发布是创建标签，标签与标记需要唯一
　　  E.1 获取提交ID git log
　　  E.2 创建标签 git  tag  1.2.3  提交ID
    F. 回退到某个历史版本
      　F.1 获取提交ID　git log
　　    F.2 回退到指定版本   git reset --hard 提交ID
    G. 使用reset命令后log是得不到充分信息的，这时我们需要使用reflog，然后再reset  git reflog
    H. 彩色git输出 git config color.ui true
    I. 查看远程分支与本地分支  git branch -a
    J. push一个指定分支名到远程分支，如果远程服务器没有这个分支则创建　git push origin <brancheName>
    K. 删除一个远程分支　　git push origin --delete <branchName>
    L. 如果使用rm误删了文件，可以通过两步恢复
　  　1. git reset HRAD 文件名
　　  2. git checkout -- 文件名
    M. 删除文件
　　  git rm 文件名    （同时删除工作目录与本地仓库的文件）
　　  git rm --cached 文件名     （删除本地仓库文件，并不影响工作目录）
    N. 改变上传地址  git remote set-url origin ssh://git@git.sailor.cn/~/WeiYu

O. 根据服务器的地址创建本地git与服务器的地址关联
　　git remote add origin ssh://lht@git_server/var/lib/scm/git/lht/test.git
尝试用qemu+gdb（or ECLIPSE-CDT）调试lab1
- [x]   

> 已调试

对于如下的代码段，请说明”：“后面的数字是什么含义
```/* Gate descriptors for interrupts and traps */struct gatedesc {    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment    unsigned gd_ss : 16;            // segment selector    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})    unsigned gd_s : 1;                // must be 0 (system)    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level    unsigned gd_p : 1;                // Present    unsigned gd_off_31_16 : 16;        // high bits of offset in segment};```
- [x]  

>   定义位域的宽度
 
对于如下的代码段，
```#define SETGATE(gate, istrap, sel, off, dpl) {            \    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \    (gate).gd_ss = (sel);                                \    (gate).gd_args = 0;                                    \    (gate).gd_rsv1 = 0;                                    \    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \    (gate).gd_s = 0;                                    \    (gate).gd_dpl = (dpl);                                \    (gate).gd_p = 1;                                    \    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \}```如果在其他代码段中有如下语句，```unsigned intr;intr=8;SETGATE(intr, 0,1,2,3);```请问执行上述指令后， intr的值是多少？
- [x]   

> 65538

请分析 [list.h](https://github.com/chyyuu/ucore_lab/blob/master/labcodes/lab2/libs/list.h)内容中大致的含义，并能include这个文件，利用其结构和功能编写一个数据结构链表操作的小C程序
- [x] 

> 

---## 开放思考题---

是否愿意挑战大实验（大实验内容来源于你的想法或老师列好的题目，需要与老师协商确定，需完成基本lab，但可不参加闭卷考试），如果有，可直接给老师email或课后面谈。
- [x]

>  
---
