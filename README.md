# WASI I/O

[WebAssembly系统接口(WebAssembly System Interface)](https://github.com/WebAssembly/WASI)API提案。

### 当前阶段（Current Phase）

WASI I/O目前处于[第3阶段(Phase 3)][Phase 3]。

[Phase 3]: https://github.com/WebAssembly/WASI/blob/main/Proposals.md#phase-3---implementation-phase-cg--wg

### 拥护者（Champions）

- Dan Gohman

### 可移植性标准（Portability Criteria）

WASI I/O必须有至少可以在Windows、macOS和Linux上通过测试套件(testsuite)的主机实现。

WASI I/O 必须至少有两个完整独立的实现。

## 目录（Table of Contents）

- [介绍（Introduction）](#介绍introduction)
- [目标（Goals）](#目标goals)
- [非目标（Non-goals）](#非目标non-goals)
- [API详解（API walk-through）](#API详解api-walk-through)
  - [用例：使用`read`/`write`从输出复制到输出（Use Case: copying from input to output using `read`/`write`）](#用例使用readwrite从输出复制到输出use-case-copying-from-input-to-output-using-readwrite)
  - [用例：使用`splice`从输入复制到输出（Use case: copying from input to output using `splice`）](#用例使用splice从输入复制到输出use-case-copying-from-input-to-output-using-splice)
  - [用例：使用`forward`复制输入到输出（Use case: copying from input to output using `forward`）](#用例使用forward复制输入到输出use-case-copying-from-input-to-output-using-forward)
- [详细设计讨论（Detailed design discussion）](#详细设计讨论detailed-design-discussion)
  - [我们是否应支持非阻塞读/写？（Should we have support for non-blocking read/write?）](#我们是否应支持非阻塞读写should-we-have-support-for-non-blocking-readwrite)
  - [为什么读/写使用u64大小？（Why do read/write use u64 sizes?）](#为什么读写使用u64大小why-do-readwrite-use-u64-sizes)
  - [当可以在循环中`splice`时，为什么还需要`forward`函数？（Why have a `forward` function when you can just `splice` in a loop?）](#当可以在循环中splice时为什么还需要forward函数why-have-a-forward-function-when-you-can-just-splice-in-a-loop)
- [项目相关方利益 & 反馈（Stakeholder Interest & Feedback）](#项目相关方利益--反馈stakeholder-interest--feedback)
- [参考文献 & 致谢（References & acknowledgements）](#参考文献--致谢references--acknowledgements)

### 介绍（Introduction）

Wasi I/O提供输入/输出流(I/O stream)抽象的API。它有两种类型，`input-stream`和`output-stream`，分别支持`read`和`write`、以及许多实用函数。

### 目标（Goals）

 - 可被wasi-libc用来实现类POSIX文件(POSIX-like file)和套接字(socket)API。
 - 支持多种不同宿主流(host stream)，包括文件(file)、套接字(socket)、管道(pipe)、字符设备(character device)等

### 非目标（Non-goals）

 - 支持异步。它将在组件模型异步设计中得到解决，这样我们能从语言绑定更紧密的集成中受益。
 - 双向流(bidirectional streams)。

### API详解（API walk-through）

#### 用例：使用`read`/`write`从输出复制到输出（Use Case: copying from input to output using `read`/`write`）

```rust
   fn copy_data(input: InputStream, output: OutputStream) -> Result<(), StreamError> {
       const BUFFER_LEN: usize = 4096;

       let wait_input = [subscribe_to_input_stream(input)];
       let wait_output = [subscribe_to_output_stream(output)];

       loop {
           let (mut data, mut eos) = input.read(BUFFER_LEN)?;

           // If we didn't get any data promptly, wait for it.
           if data.len() == 0 {
               let _ = poll_list(&wait_input[..]);
               (data, eos) = input.read(BUFFER_LEN)?;
           }

           let mut remaining = &data[..];
           while !remaining.is_empty() {
               let mut num_written = output.write(remaining)?;

               // If we didn't put any data promptly, wait for it.
               if num_written == 0 {
                   let _ = poll_list(&wait_output[..]);
                   num_written = output.write(remaining)?;
               }

               remaining = &remaining[num_written..];
           }
           if eos {
               break;
           }
       }
       Ok(())
   }
}
```

#### 用例：使用`splice`从输入复制到输出（Use case: copying from input to output using `splice`）

```rust
   fn copy_data(input: InputStream, output: OutputStream) -> Result<(), StreamError> {
       let wait_input = [subscribe_to_input_stream(input)];

       loop {
           let (num_copied, eos) = output.splice(input, u64::MAX)?;
           if eos {
               break;
           }

           // If we didn't get any data promptly, wait for it.
           if num_copied == 0 {
               let _ = poll_list(&wait_input[..]);
           }
       }
       Ok(())
   }
}
```

#### 用例：使用`forward`复制输入到输出（Use case: copying from input to output using `forward`）

```rust
   fn copy_data(input: InputStream, output: OutputStream) -> Result<(), StreamError> {
       output.forward(input)?;
       Ok(())
   }
}
```

### 详细设计讨论（Detailed design discussion）

#### 我们是否应支持非阻塞读/写？（Should we have support for non-blocking read/write?）

这可能是我们需要重新考虑的事情，但目前，非阻塞流(non-blocking stream)的工作方式是它们执行读取或写入比请求的字节数更少。

#### 为什么读/写使用u64大小？（Why do read/write use u64 sizes?）

这是为了使API独立于调用者的地址空间大小。仍然建议被调用者避免使用大于其实例能够分配的大小。

#### 当可以在循环中`splice`时，为什么还需要`forward`函数？（Why have a `forward` function when you can just `splice` in a loop?）

这似乎是一个常见的用例，而`forward`以一种非常简单的方式解决了它。

### 项目相关方利益 & 反馈（Stakeholder Interest & Feedback）

Wasi-is是wasi-filesystem、wasi-sockets和wasi-http的依赖项，也是WASI Preview 2的基础部分。

### 参考文献 & 致谢（References & acknowledgements）

非常感谢以下人员提供的宝贵反馈和建议：

- 感谢Luke Wagner的许多设计功能以及为此处的设计提供参考的组件模型异步流的设计。
- 感谢Calvin Prewitt提出在该API包含`forward`函数的想法。
