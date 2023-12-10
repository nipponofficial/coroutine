# 计算机系统概论 Coroutine Lab Report
### 李文赢 计05 2020080108

## Task 1
## 基本要求
### [1] 调度
在 serial_execute_all() 函数实现循环遍历所有协程，以及检查执行状态，直到所有协程都完成，代码修改见 Task2

### [2] Swap
```assembly
// ./lib/context.S
coroutine_switch:
    # 保存 callee-saved 寄存器到 %rdi 指向的上下文
    movq %rsp, 64(%rdi)
    movq %rbx, 72(%rdi)
    movq %rbp, 80(%rdi)
    movq %r12, 88(%rdi)
    movq %r13, 96(%rdi)
    movq %r14, 104(%rdi)
    movq %r15, 112(%rdi)
    # 保存的上下文中 rip 指向 ret 指令的地址（.coroutine_ret）
    leaq .coroutine_ret(%rip), %rax
    movq %rax, 120(%rdi)
    # 从 %rsi 指向的上下文恢复 callee-saved 寄存器
    movq 64(%rsi), %rsp
    movq 72(%rsi), %rbx
    movq 80(%rsi), %rbp
    movq 88(%rsi), %r12
    movq 96(%rsi), %r13
    movq 104(%rsi), %r14
    movq 112(%rsi), %r15
    # 最后 jmpq 到上下文保存的 rip
    movq 120(%rsi), %rax
    jmpq *%rax
```

### [3] 主动切出

```C++
/* ./inc/common.h */
void yield() {
  if (!g_pool->is_parallel) {
    auto context = g_pool->coroutines[g_pool->context_id];

    // 调用 coroutine_switch 切换到 coroutine_pool 上下文
    coroutine_switch(context->callee_registers, context->caller_registers);
  }
}
```

```C++
/* ./inc/context.h */
  virtual void resume() {
    // 调用 coroutine_switch
    coroutine_switch(caller_registers, callee_registers);
    // 在汇编中保存 callee-saved 寄存器，设置协程函数栈帧，然后将 rip 恢复到协程 yield 之后所需要执行的指令地址。
  }
```

## 额外要求
### 1. 绘制协程切换时，栈的变化过程


### 2. 分析源代码，解释协程是如何跑起来的，包括 coroutine_entry 和 coroutine_main 函数以及初始的协程状态；
