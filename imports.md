<h1><a name="imports">World imports</a></h1>
<ul>
<li>Imports:
<ul>
<li>interface <a href="#wasi_io_error_0_2_0"><code>wasi:io/error@0.2.0</code></a></li>
<li>interface <a href="#wasi_io_poll_0_2_0"><code>wasi:io/poll@0.2.0</code></a></li>
<li>interface <a href="#wasi_io_streams_0_2_0"><code>wasi:io/streams@0.2.0</code></a></li>
</ul>
</li>
</ul>
<h2><a name="wasi_io_error_0_2_0"></a>Import interface wasi:io/error@0.2.0</h2>
<hr />
<h3>Types</h3>
<h4><a name="error"></a><code>resource error</code></h4>
<p>表示某些错误信息的资源。</p>
<p>此资源(resource)提供的唯一方法是<code>to-debug-string</code>，它提供了一些关于错误的人类可读信息。</p>
<p>在<code>wasi:io</code>包中，此资源(resource)通过<code>wasi:io/streams/stream-error</code>类型返回。</p>
<p>为了提供更具体的错误信息，其他接口(interface)可能提供函数来进一步地将错误“向下转换(downcast)”为更具体的错误信息。
例如，从文件系统类型派生的流返回的[错误][error]，可以使用文件系统自己的错误码类型来描述，
使用函数<code>wasi:filesystem/types/filesystem-error-code</code>，
其接受一个<code>borrow&lt;error&gt;</code>参数并返回<code>option&lt;wasi:filesystem/types/error-code&gt;</code>。</p>
<h2>可以将<a href="#error"><code>error</code></a>“向下转换”为更具体类型的函数集是开放的。</h2>
<h3>Functions</h3>
<h4><a name="method_error_to_debug_string"></a><code>[method]error.to-debug-string: func</code></h4>
<p>返回一个适合帮助人类调试此错误的字符串。</p>
<p>警告：返回的字符串不应被机器使用！
它可能会随平台、宿主或其他实现细节而变化。
解析这个字符串是严重的平台兼容性隐患。</p>
<h5>Params</h5>
<ul>
<li><a name="method_error_to_debug_string.self"></a><code>self</code>: borrow&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_error_to_debug_string.0"></a> <code>string</code></li>
</ul>
<h2><a name="wasi_io_poll_0_2_0"></a>Import interface wasi:io/poll@0.2.0</h2>
<p>轮询API旨在让用户同时等待多个句柄上的输入/输出(I/O)事件</p>
<hr />
<h3>Types</h3>
<h4><a name="pollable"></a><code>resource pollable</code></h4>
<h2><a href="#pollable"><code>pollable</code></a>表示一个可能已就绪(ready)或未就绪的单个输入/输出(I/O)事件。</h2>
<h3>Functions</h3>
<h4><a name="method_pollable_ready"></a><code>[method]pollable.ready: func</code></h4>
<p>返回pollable(可轮询项)的就绪状态(readiness)。此函数永不阻塞。</p>
<p>当pollable(可轮询项)已就绪返回<code>true</code>，否则返回<code>false</code>。</p>
<h5>Params</h5>
<ul>
<li><a name="method_pollable_ready.self"></a><code>self</code>: borrow&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_pollable_ready.0"></a> <code>bool</code></li>
</ul>
<h4><a name="method_pollable_block"></a><code>[method]pollable.block: func</code></h4>
<p>如果pollable(可轮询项)已就绪，<code>block</code>立即返回，否则阻塞至准备就绪。</p>
<p>此函数等同于在只包含这个pollable(可轮询项)的列表上调用<code>poll.poll</code>。</p>
<h5>Params</h5>
<ul>
<li><a name="method_pollable_block.self"></a><code>self</code>: borrow&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h4><a name="poll"></a><code>poll: func</code></h4>
<p>对一组可轮询项进行轮询以完成。</p>
<p>此函数接受一个pollable列表，这些可轮询项(pollable)标识其关注的输入/输出(I/O)源，
并等待直到一个或多个事件准备就绪进行输入/输出(I/O)。</p>
<p>返回值<code>list&lt;u32&gt;</code>包含了参数列表中一个或多个输入/输出(I/O)已就绪的句柄索引。</p>
<p>此函数捕获两者任何一个：</p>
<ul>
<li>list为空，或者：</li>
<li>list包含的元素超过<code>u32</code>值的可索引范围。</li>
</ul>
<p>超时(timeout)可以通过将wasi-clocks API的pollable添加至list实现。</p>
<p>此函数不返回<code>result</code>；轮询自身不进行任何输入/输出(I/O)，因此不会失败。
如果pollable标识的任何输入/输出(I/O)源有错误，则通过将源标记为已就绪I/O来表示</p>
<h5>Params</h5>
<ul>
<li><a name="poll.in"></a><code>in</code>: list&lt;borrow&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="poll.0"></a> list&lt;<code>u32</code>&gt;</li>
</ul>
<h2><a name="wasi_io_streams_0_2_0"></a>Import interface wasi:io/streams@0.2.0</h2>
<p>WASI I/O是输出/输出(I/O)抽象API，目前专注于提供stream类型。</p>
<p>未来，组件模型预计会添加内置(built-in)stream类型；当其发生时，它将应包含此API。</p>
<hr />
<h3>Types</h3>
<h4><a name="error"></a><code>type error</code></h4>
<p><a href="#error"><a href="#error"><code>error</code></a></a></p>
<p>
#### <a name="pollable"></a>`type pollable`
[`pollable`](#pollable)
<p>
#### <a name="stream_error"></a>`variant stream-error`
<p>输入流(input-stream)和输出流(output-stream)操作的错误</p>
<h5>Variant Cases</h5>
<ul>
<li>
<p><a name="stream_error.last_operation_failed"></a><code>last-operation-failed</code>: own&lt;<a href="#error"><a href="#error"><code>error</code></a></a>&gt;</p>
<p>在完成之前的最后一次失败操作（write或flutsh）。
<p>更多信息可以在<a href="#error"><code>error</code></a>载荷中获取。</p>
</li>
<li>
<p><a name="stream_error.closed"></a><code>closed</code></p>
<p>流已关闭：流将不再接受任何输入。关闭的输出流将在后续的所有操作中返回此错误。
</li>
</ul>
<h4><a name="input_stream"></a><code>resource input-stream</code></h4>
<p>输入字节流(bytestream)。</p>
<p><a href="#input_stream"><code>input-stream</code></a>在底层架构上实际情况是<em>非阻塞的</em>。
I/O操作总是立即返回；如果立即可用的字节数少于请求的，则其返回立即获得的字节数，甚至可能为零。
为了等待可用数据，使用<code>subscribe</code>函数获取<a href="#pollable"><code>pollable</code></a>，其可用<code>wasi:io/poll</code>进行轮询。</p>
<h4><a name="output_stream"></a><code>resource output-stream</code></h4>
<p>输出字节流。</p>
<h2><a href="#output_stream"><code>output-stream</code></a>在底层架构上实际情况是<em>非阻塞的</em>。
除非另有说明，I/O操作也总是立即返回，返回可立即写入的字节数，甚至可能为零。
为了等待流准备就绪接受数据，使用<code>subscribe</code>函数获取<a href="#pollable"><code>pollable</code></a>，其可用<code>wasi:io/poll</code>轮询。</h2>
<h3>Functions</h3>
<h4><a name="method_input_stream_read"></a><code>[method]input-stream.read: func</code></h4>
<p>从流中执行非阻塞读取</p>
<p>当<code>read</code>的源数据为二进制时，将原样返回源字节。
当已知<code>read</code>的源数据实现为文本时，将返回包含该文本的UTF-8编码字节。</p>
<p>当成功时此函数返回包含读取数据的字节列表(<code>list&lt;u8&gt;</code>)。
返回的list最多包含<code>len</code>个字节；其返回可能少于请求，但不会多。
当前没有可读取的字节时，则list为空。
当有更多可读取的字节时，则<code>subscribe</code>提供的pollable(可轮询对象)将ready(准备就绪)。</p>
<p>此函数在操作遇到错误时会失败并返回<a href="#stream_error"><code>stream-error</code></a>，错误为<code>last-operation-failed</code>，
或者当stream(流)被关闭时，错误为<code>closed</code>。</p>
<p>当调用者传递<code>len</code>为0时，其代表请求读取0字节。
如果stream(流)仍然为open(打开的)，此调用应成功并返回一个空list，
否则(stream已关闭)返回<code>closed</code>失败。</p>
<p>参数<code>len</code>为<code>u64</code>，它可以表示一个在wasm32中无法分配的u8列表，或者不希望由被调用者分配作为返回值。
当有更多可读取字节时，被调用者可以返回一个小于<code>len</code>大小的字节list(列表)。</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream_read.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream_read.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream_read.0"></a> result&lt;list&lt;<code>u8</code>&gt;, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream_blocking_read"></a><code>[method]input-stream.blocking-read: func</code></h4>
<p>从流中读取字节，阻塞直到至少可以读取一个字节。
除了阻塞，行为与<code>read</code>相同。</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream_blocking_read.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream_blocking_read.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream_blocking_read.0"></a> result&lt;list&lt;<code>u8</code>&gt;, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream_skip"></a><code>[method]input-stream.skip: func</code></h4>
<p>从流中跳过若干字节。返回跳过的字节数。</p>
<p>行为与<code>read</code>相同，除了不返回字节list，
而是返回从流中消耗的字节数。</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream_skip.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream_skip.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream_skip.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream_blocking_skip"></a><code>[method]input-stream.blocking-skip: func</code></h4>
<p>从流中跳过若干字节，阻塞直到至少可以跳过一个字节。
除了阻塞行为，与<code>skip</code>相同。</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream_blocking_skip.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_input_stream_blocking_skip.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream_blocking_skip.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_input_stream_subscribe"></a><code>[method]input-stream.subscribe: func</code></h4>
<p>创建<a href="#pollable"><code>pollable</code></a>(可轮询对象)，它在指定的流有可读取字节或流的另一端被关闭时解析。
创建的<a href="#pollable"><code>pollable</code></a>是<a href="#input_stream"><code>input-stream</code></a>的子资源(child resource)。
若在此函数创建导出的<a href="#pollable"><code>pollable</code></a>均被释放之前丢弃<a href="#input_stream"><code>input-stream</code></a>，实现可能会出错。</p>
<h5>Params</h5>
<ul>
<li><a name="method_input_stream_subscribe.self"></a><code>self</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_input_stream_subscribe.0"></a> own&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_check_write"></a><code>[method]output-stream.check-write: func</code></h4>
<p>检查写入就绪状态。此函数永远不会阻塞。</p>
<p>返回允许下次调用<code>write</code>的字节数，或返回一个错误。
使用超过函数允许的字节数调用<code>write</code>将会出错。</p>
<p>当函数返回0字节时，<code>subscribe</code>的pollable(可轮询对象)
将会在此函数将报告至少1字节或错误时，变为就绪状态。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_check_write.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_check_write.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_write"></a><code>[method]output-stream.write: func</code></h4>
<p>执行写入。此函数永远不会阻塞。</p>
<p>当<code>write</code>的目标为二进制数据时，来自<code>contents</code>的字节将原样写入。
当已知<code>write</code>的目标实现为文本时，<code>contents</code>的字节将从UTF-8转码为目标编码后写入。</p>
<p>前提条件：check-write返回允许的Ok(n)，因此contents的长度应小于或等于n。
否则，此函数将出错。</p>
<p>如果自上次调用check-write获取许可后流已关闭，则此函数在不写入时返回Err(closed)。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_write.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_write.contents"></a><code>contents</code>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_write.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_blocking_write_and_flush"></a><code>[method]output-stream.blocking-write-and-flush: func</code></h4>
<p>执行最多4096字节的写入，然后刷新(flush)流。
阻塞直到这些操作全部完成，或者出现错误。</p>
<p>此为<code>check-write</code>、<code>subscribe</code>、<code>write</code>、和<code>flush</code>的便携包装，
其伪代码实现如下：</p>
<pre><code class="language-text">let pollable = this.subscribe();
while !contents.is_empty() {
  // 等待流变为可写入
  pollable.block();
  let Ok(n) = this.check-write(); // 省略错误处理
  let len = min(n, contents.len());
  let (chunk, rest) = contents.split_at(len);
  this.write(chunk);              // 省略错误处理
  contents = rest;
}
this.flush();
// 等待`flush`完成
pollable.block();
// 检查`flush`过程中出现的任何错误
let _ = this.check-write();         // 省略错误处理
</code></pre>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_blocking_write_and_flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_blocking_write_and_flush.contents"></a><code>contents</code>: list&lt;<code>u8</code>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_blocking_write_and_flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_flush"></a><code>[method]output-stream.flush: func</code></h4>
<p>请求刷新缓冲输出。此函数永远不会阻塞。</p>
<p>其告知output-stream调用者希望刷新任何已缓冲的输出。
预期将刷新在此调用之前所有<code>write</code>已传递的输出</p>
<p>调用此函数后，在刷新完成之前<a href="#output_stream"><code>output-stream</code></a>将不接受任何写入（<code>check-write</code> 将返回<code>ok(0)</code>）。
当刷新完成且流可以接受更多写入时<code>subscribe</code>的pollable(可轮询对象)将变为就绪状态。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_blocking_flush"></a><code>[method]output-stream.blocking-flush: func</code></h4>
<p>请求刷新缓冲输出，并且阻塞直到刷新完成且流准备就绪再次写入。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_blocking_flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_blocking_flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_subscribe"></a><code>[method]output-stream.subscribe: func</code></h4>
<p>创建<a href="#pollable"><code>pollable</code></a>，其将在output-stream准备就绪更多写入或发生错误时解析(resolve)。
当pollable(可轮询对象)已就绪时，<code>check-write</code>将返回<code>ok(n)</code>，其中n&gt;0，或者返回一个错误。</p>
<p>如果流已关闭，此pollable总是立即准备就绪。</p>
<p>创建的<a href="#pollable"><code>pollable</code></a>是<a href="#output_stream"><code>output-stream</code></a>子资源(child resource)。
若在此函数创建导出的<a href="#pollable"><code>pollable</code></a>均被释放之前丢弃<a href="#output_stream"><code>output-stream</code></a>，实现可能会出错。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_subscribe.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_subscribe.0"></a> own&lt;<a href="#pollable"><a href="#pollable"><code>pollable</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_write_zeroes"></a><code>[method]output-stream.write-zeroes: func</code></h4>
<p>向流写入zeroes</p>
<p>此应与<code>write</code>的使用方式完全相同，具有完全相同的前提条件（必须先使用check-write），
但不传递字节列表(list)，而是传递应写入的zero-bytes数。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_write_zeroes.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_write_zeroes.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_write_zeroes.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_blocking_write_zeroes_and_flush"></a><code>[method]output-stream.blocking-write-zeroes-and-flush: func</code></h4>
<p>执行最多4096 zeroes的写入，然后刷新(flush)流。
阻塞直到这些操作全部完成，或者出现错误。</p>
<p>此为<code>check-write</code>、<code>subscribe</code>、<code>write-zeroes</code>、和<code>flush</code>的便携包装，
其伪代码实现如下：</p>
<pre><code class="language-text">let pollable = this.subscribe();
while num_zeroes != 0 {
  // Wait for the stream to become writable
  pollable.block();
  let Ok(n) = this.check-write(); // 省略错误处理
  let len = min(n, num_zeroes);
  this.write-zeroes(len);         // 省略错误处理
  num_zeroes -= len;
}
this.flush();
// 等待`flush`完成
pollable.block();
// 检查`flush`过程中出现的任何错误
let _ = this.check-write();         // 省略错误处理
</code></pre>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_blocking_write_zeroes_and_flush.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_blocking_write_zeroes_and_flush.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_blocking_write_zeroes_and_flush.0"></a> result&lt;_, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_splice"></a><code>[method]output-stream.splice: func</code></h4>
<p>从一个流中读取并写入到另一个流。</p>
<p>splice的行为等同于：</p>
<ol>
<li>调用<a href="#output_stream"><code>output-stream</code></a>上的<code>check-write</code></li>
<li>调用<a href="#input_stream"><code>input-stream</code></a>上的<code>read</code>，其参数取<code>check-write</code>准许长度和<code>splice</code>提供的<code>len</code>两者较小之一</li>
<li>调用<a href="#output_stream"><code>output-stream</code></a>上的<code>write</code>，其参数为read的数据。</li>
</ol>
<p>调用<code>check-write</code>、<code>read</code>、或<code>write</code>期间报告的错误，都会终止splice并报告该错误</p>
<p>此函数返回传输的字节数；其可能小于<code>len</code>。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_splice.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_splice.src"></a><code>src</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_splice.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_splice.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
<h4><a name="method_output_stream_blocking_splice"></a><code>[method]output-stream.blocking-splice: func</code></h4>
<p>从一个流中读取并写入到另一个流，有阻塞。</p>
<p>其类似于<code>splice</code>，除了在执行<code>splice</code>会阻塞，
直到<a href="#output_stream"><code>output-stream</code></a>准备就绪写入，且<a href="#input_stream"><code>input-stream</code></a>准备就绪读取。</p>
<h5>Params</h5>
<ul>
<li><a name="method_output_stream_blocking_splice.self"></a><code>self</code>: borrow&lt;<a href="#output_stream"><a href="#output_stream"><code>output-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_blocking_splice.src"></a><code>src</code>: borrow&lt;<a href="#input_stream"><a href="#input_stream"><code>input-stream</code></a></a>&gt;</li>
<li><a name="method_output_stream_blocking_splice.len"></a><code>len</code>: <code>u64</code></li>
</ul>
<h5>Return values</h5>
<ul>
<li><a name="method_output_stream_blocking_splice.0"></a> result&lt;<code>u64</code>, <a href="#stream_error"><a href="#stream_error"><code>stream-error</code></a></a>&gt;</li>
</ul>
