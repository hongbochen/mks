---
title: Dalvik字节码 
tags: Dalvik,字节码
grammar_cjkRuby: true
---

## 总体设计

<hr />

 - 机器模型和调用约定是近似模仿常见的真实的架构和C风格调用约定。

  - 机器是基于寄存器的，并且框架在被创建的时候是固定大小的。每一框架包含一个特定数量的寄存器（由函数指定）和一些需要执行该函数的附属的数据，例如（但不限制在这些）程序计数器和包含该方法的`.dex`文件的引用。

  - 当用于位值的时候（例如整数和浮点数），寄存器被认为是32位宽度。相邻的寄存器对被用于64位值。对于寄存器对没有对齐要求。

  - 当用于对象引用的时候，寄存器被认为有足够的宽度来维持整个引用。

  - 在按位表示方面，(Object)null == (int)0。

  - 一个方法的N个参数顺序排列在函数调用框架的后N个寄存器中。长参数消耗两个寄存器。实例函数被传送一个`this`引用作为他的第一个参数。

 - 在指令流中的存储单元是一个16位的无符号数。在一些指令中的一些为被忽略或者必须为０。

 - 指令不能被平白限制到一个特定类型。例如，指令在没有解释下移动32位寄存器值不必指定他们移动了整数还是浮点数。

 - 有针对字符串，属性，类型和函数引用的分离的枚举和索引的连续的空间。

 - 按位文字数据在指令流中代表内嵌的。

 - 在实际情况中，因为一个函数需要使用超过16个寄存器是不寻常的，并且因为需要超过8个寄存器是相当普遍的，所以很多指令被限制只能寻址前16个寄存器。当合理的可能的情况下，指令允许寻址达到前256个寄存器。除此之外，有些变型指令允许更多数量的寄存器，包括一对捕捉所有`move`指令的可以寻址寄存器的范围是`v0-v65535`。当一个指令变型不能够寻址期望的寄存器，他期望的是，寄存器的内容从原始的寄存器移动到了一个低寄存器上（在操作前）并且/或者从一个低结果寄存器移动到了一个高寄存器上（在操作之后）。

 - 有一些"伪指令"，被用于持有可变长度的数据的有效载荷，他们被常规指令引用（例如，`fill-array-data`）。在执行的正常流中，这样的指令是决不能遇到的。除此之外，指令必须位于偶数字节偏移（也就是。4字节对齐）。为了满足这个要求，dex生成工具必须生成一个额外的`nop`指令作为一个空格，如果这样一个指令没有被对齐的话。最终，虽然不是必须的，但是大多数工具将会选择在函数的最后提供这些指令，否则可能会需要额外的指令在他们周围分支。

 - 在安装到一个运行的系统中的时候，一些指令可能被改变，改变他们的格式，作为一个安装时静态链接优化。一旦链接被知晓，这将会获得更快的执行速度。我们将在后面学习指令格式的相关知识用于"suggested variants"。"suggested"被故意使用。实施这些并不是强制性的。

 - 人类语法和记忆：

  - 用于参数的先目的地后源头的排序

  - 一些操作码会有消除歧义的名称后缀来显示他们操作的类型：



1. 通用型32位操作码是不被标记的。

2. 通用型64位操作码带有后缀`-wide`。

3. 指定类型的操作码使用他们的类型（或者是一个直接的缩略词）作为后缀，例如`-boolean -byte -char -short -int -long -float -double -object -string -class -void`。

  - 一些操作码有一个消除歧义的后缀去区分那么否则会有相同操作的指令，这些指令有不同的指令布局或选项。这些后缀与主名称之间使用一个斜杠"/"进行分离，并且主要存在与使生成或解释可执行程序的代码中具有静态常亮的一对一映射的地方（这是为了降低人类歧义）。

  - 在这里的解释中，一个值的宽度(一个常亮的范围或者是可能寻址的寄存器数量)被每四位宽度一个字符的使用所强调了。

  - 例如，在指令`move-wide/from 16 vAA,vBBBB`：



1. "move"是基础的操作码，代表基本的操作(move是一个寄存器值)。

2. "wide"是名称后缀，代表他操作宽(64位)数据。

3. "from 16"是操作码后缀，代表拥有一个16位的寄存器引用的变量作为源。

4. "vAA"是目的寄存器(由操作指出；规则就是目的参数永远第一个出现)，目的寄存器的范围是`v0-v255`。

5. "vBBBB"是源寄存器，他的范围是`v0-v65535`。

 - 在后面的文章"指令格式文档"中，我们将会学习关于变量指令格式的细节，还有一些关于操作码语法的细节。

 - 后面文章".dex文件格式文档"将会学习更多的关于字节码与大环境的适应。



## 字节码集合的总结

<hr / >

<table class="instruc">

<thead>

<tr>

  <th>Op &amp; Format</th>

  <th>Mnemonic / Syntax</th>

  <th>Arguments</th>

  <th>Description</th>

</tr>

</thead>

<tbody>

<tr>

  <td>00 10x</td>

  <td>nop</td>

  <td>&nbsp;</td>

  <td>Waste cycles.

    <p class="note"><strong>Note:</strong>

    Data-bearing pseudo-instructions are tagged with this opcode, in which

    case the high-order byte of the opcode unit indicates the nature of

    the data. See "<code>packed-switch-payload</code> Format",

    "<code>sparse-switch-payload</code> Format", and

    "<code>fill-array-data-payload</code> Format" below.</p>

  </td>

</tr>

<tr>

  <td>01 12x</td>

  <td>move vA, vB</td>

  <td><code>A:</code> destination register (4 bits)<br>

    <code>B:</code> source register (4 bits)</td>

  <td>Move the contents of one non-object register to another.</td>

</tr>

<tr>

  <td>02 22x</td>

  <td>move/from16 vAA, vBBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> source register (16 bits)</td>

  <td>Move the contents of one non-object register to another.</td>

</tr>

<tr>

  <td>03 32x</td>

  <td>move/16 vAAAA, vBBBB</td>

  <td><code>A:</code> destination register (16 bits)<br>

    <code>B:</code> source register (16 bits)</td>

  <td>Move the contents of one non-object register to another.</td>

</tr>

<tr>

  <td>04 12x</td>

  <td>move-wide vA, vB</td>

  <td><code>A:</code> destination register pair (4 bits)<br>

    <code>B:</code> source register pair (4 bits)</td>

  <td>Move the contents of one register-pair to another.

    <p class="note"><strong>Note:</strong>

    It is legal to move from <code>v<i>N</i></code> to either

    <code>v<i>N-1</i></code> or <code>v<i>N+1</i></code>, so implementations

    must arrange for both halves of a register pair to be read before

    anything is written.</p>

  </td>

</tr>

<tr>

  <td>05 22x</td>

  <td>move-wide/from16 vAA, vBBBB</td>

  <td><code>A:</code> destination register pair (8 bits)<br>

    <code>B:</code> source register pair (16 bits)</td>

  <td>Move the contents of one register-pair to another.

    <p class="note"><strong>Note:</strong>

    Implementation considerations are the same as <code>move-wide</code>,

    above.</p>

  </td>

</tr>

<tr>

  <td>06 32x</td>

  <td>move-wide/16 vAAAA, vBBBB</td>

  <td><code>A:</code> destination register pair (16 bits)<br>

    <code>B:</code> source register pair (16 bits)</td>

  <td>Move the contents of one register-pair to another.

    <p class="note"><strong>Note:</strong>

    Implementation considerations are the same as <code>move-wide</code>,

    above.</p>

  </td>

</tr>

<tr>

  <td>07 12x</td>

  <td>move-object vA, vB</td>

  <td><code>A:</code> destination register (4 bits)<br>

    <code>B:</code> source register (4 bits)</td>

  <td>Move the contents of one object-bearing register to another.</td>

</tr>

<tr>

  <td>08 22x</td>

  <td>move-object/from16 vAA, vBBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> source register (16 bits)</td>

  <td>Move the contents of one object-bearing register to another.</td>

</tr>

<tr>

  <td>09 32x</td>

  <td>move-object/16 vAAAA, vBBBB</td>

  <td><code>A:</code> destination register (16 bits)<br>

    <code>B:</code> source register (16 bits)</td>

  <td>Move the contents of one object-bearing register to another.</td>

</tr>

<tr>

  <td>0a 11x</td>

  <td>move-result vAA</td>

  <td><code>A:</code> destination register (8 bits)</td>

  <td>Move the single-word non-object result of the most recent

    <code>invoke-<i>kind</i></code> into the indicated register.

    This must be done as the instruction immediately after an

    <code>invoke-<i>kind</i></code> whose (single-word, non-object) result

    is not to be ignored; anywhere else is invalid.</td>

</tr>

<tr>

  <td>0b 11x</td>

  <td>move-result-wide vAA</td>

  <td><code>A:</code> destination register pair (8 bits)</td>

  <td>Move the double-word result of the most recent

    <code>invoke-<i>kind</i></code> into the indicated register pair.

    This must be done as the instruction immediately after an

    <code>invoke-<i>kind</i></code> whose (double-word) result

    is not to be ignored; anywhere else is invalid.</td>

</tr>

<tr>

  <td>0c 11x</td>

  <td>move-result-object vAA</td>

  <td><code>A:</code> destination register (8 bits)</td>

  <td>Move the object result of the most recent <code>invoke-<i>kind</i></code>

    into the indicated register. This must be done as the instruction

    immediately after an <code>invoke-<i>kind</i></code> or

    <code>filled-new-array</code>

    whose (object) result is not to be ignored; anywhere else is invalid.</td>

</tr>

<tr>

  <td>0d 11x</td>

  <td>move-exception vAA</td>

  <td><code>A:</code> destination register (8 bits)</td>

  <td>Save a just-caught exception into the given register. This must

    be the first instruction of any exception handler whose caught

    exception is not to be ignored, and this instruction must <i>only</i>

    ever occur as the first instruction of an exception handler; anywhere

    else is invalid.</td>

</tr>

<tr>

  <td>0e 10x</td>

  <td>return-void</td>

  <td>&nbsp;</td>

  <td>Return from a <code>void</code> method.</td>

</tr>

<tr>

  <td>0f 11x</td>

  <td>return vAA</td>

  <td><code>A:</code> return value register (8 bits)</td>

  <td>Return from a single-width (32-bit) non-object value-returning

    method.

  </td>

</tr>

<tr>

  <td>10 11x</td>

  <td>return-wide vAA</td>

  <td><code>A:</code> return value register-pair (8 bits)</td>

  <td>Return from a double-width (64-bit) value-returning method.</td>

</tr>

<tr>

  <td>11 11x</td>

  <td>return-object vAA</td>

  <td><code>A:</code> return value register (8 bits)</td>

  <td>Return from an object-returning method.</td>

</tr>

<tr>

  <td>12 11n</td>

  <td>const/4 vA, #+B</td>

  <td><code>A:</code> destination register (4 bits)<br>

    <code>B:</code> signed int (4 bits)</td>

  <td>Move the given literal value (sign-extended to 32 bits) into

    the specified register.</td>

</tr>

<tr>

  <td>13 21s</td>

  <td>const/16 vAA, #+BBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> signed int (16 bits)</td>

  <td>Move the given literal value (sign-extended to 32 bits) into

    the specified register.</td>

</tr>

<tr>

  <td>14 31i</td>

  <td>const vAA, #+BBBBBBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> arbitrary 32-bit constant</td>

  <td>Move the given literal value into the specified register.</td>

</tr>

<tr>

  <td>15 21h</td>

  <td>const/high16 vAA, #+BBBB0000</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> signed int (16 bits)</td>

  <td>Move the given literal value (right-zero-extended to 32 bits) into

    the specified register.</td>

</tr>

<tr>

  <td>16 21s</td>

  <td>const-wide/16 vAA, #+BBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> signed int (16 bits)</td>

  <td>Move the given literal value (sign-extended to 64 bits) into

    the specified register-pair.</td>

</tr>

<tr>

  <td>17 31i</td>

  <td>const-wide/32 vAA, #+BBBBBBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> signed int (32 bits)</td>

  <td>Move the given literal value (sign-extended to 64 bits) into

    the specified register-pair.</td>

</tr>

<tr>

  <td>18 51l</td>

  <td>const-wide vAA, #+BBBBBBBBBBBBBBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> arbitrary double-width (64-bit) constant</td>

  <td>Move the given literal value into

    the specified register-pair.</td>

</tr>

<tr>

  <td>19 21h</td>

  <td>const-wide/high16 vAA, #+BBBB000000000000</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> signed int (16 bits)</td>

  <td>Move the given literal value (right-zero-extended to 64 bits) into

    the specified register-pair.</td>

</tr>

<tr>

  <td>1a 21c</td>

  <td>const-string vAA, string@BBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> string index</td>

  <td>Move a reference to the string specified by the given index into the

    specified register.</td>

</tr>

<tr>

  <td>1b 31c</td>

  <td>const-string/jumbo vAA, string@BBBBBBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> string index</td>

  <td>Move a reference to the string specified by the given index into the

    specified register.</td>

</tr>

<tr>

  <td>1c 21c</td>

  <td>const-class vAA, type@BBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> type index</td>

  <td>Move a reference to the class specified by the given index into the

    specified register. In the case where the indicated type is primitive,

    this will store a reference to the primitive type's degenerate

    class.</td>

</tr>

<tr>

  <td>1d 11x</td>

  <td>monitor-enter vAA</td>

  <td><code>A:</code> reference-bearing register (8 bits)</td>

  <td>Acquire the monitor for the indicated object.</td>

</tr>

<tr>

  <td>1e 11x</td>

  <td>monitor-exit vAA</td>

  <td><code>A:</code> reference-bearing register (8 bits)</td>

  <td>Release the monitor for the indicated object.

    <p class="note"><strong>Note:</strong>

    If this instruction needs to throw an exception, it must do

    so as if the pc has already advanced past the instruction.

    It may be useful to think of this as the instruction successfully

    executing (in a sense), and the exception getting thrown <i>after</i>

    the instruction but <i>before</i> the next one gets a chance to

    run. This definition makes it possible for a method to use

    a monitor cleanup catch-all (e.g., <code>finally</code>) block as

    the monitor cleanup for that block itself, as a way to handle the

    arbitrary exceptions that might get thrown due to the historical

    implementation of <code>Thread.stop()</code>, while still managing

    to have proper monitor hygiene.</p>

  </td>

</tr>

<tr>

  <td>1f 21c</td>

  <td>check-cast vAA, type@BBBB</td>

  <td><code>A:</code> reference-bearing register (8 bits)<br>

    <code>B:</code> type index (16 bits)</td>

  <td>Throw a <code>ClassCastException</code> if the reference in the

    given register cannot be cast to the indicated type.

    <p class="note"><strong>Note:</strong> Since <code>A</code> must always be a reference

    (and not a primitive value), this will necessarily fail at runtime

    (that is, it will throw an exception) if <code>B</code> refers to a

    primitive type.</p>

  </td>

</tr>

<tr>

  <td>20 22c</td>

  <td>instance-of vA, vB, type@CCCC</td>

  <td><code>A:</code> destination register (4 bits)<br>

    <code>B:</code> reference-bearing register (4 bits)<br>

    <code>C:</code> type index (16 bits)</td>

  <td>Store in the given destination register <code>1</code>

    if the indicated reference is an instance of the given type,

    or <code>0</code> if not.

    <p class="note"><strong>Note:</strong> Since <code>B</code> must always be a reference

    (and not a primitive value), this will always result

    in <code>0</code> being stored if <code>C</code> refers to a primitive

    type.</p></td>

</tr>

<tr>

  <td>21 12x</td>

  <td>array-length vA, vB</td>

  <td><code>A:</code> destination register (4 bits)<br>

    <code>B:</code> array reference-bearing register (4 bits)</td>

  <td>Store in the given destination register the length of the indicated

    array, in entries</td>

</tr>

<tr>

  <td>22 21c</td>

  <td>new-instance vAA, type@BBBB</td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> type index</td>

  <td>Construct a new instance of the indicated type, storing a

    reference to it in the destination. The type must refer to a

    non-array class.</td>

</tr>

<tr>

  <td>23 22c</td>

  <td>new-array vA, vB, type@CCCC</td>

  <td><code>A:</code> destination register (4 bits)<br>

    <code>B:</code> size register<br>

    <code>C:</code> type index</td>

  <td>Construct a new array of the indicated type and size. The type

    must be an array type.</td>

</tr>

<tr>

  <td>24 35c</td>

  <td>filled-new-array {vC, vD, vE, vF, vG}, type@BBBB</td>

  <td>

    <code>A:</code> array size and argument word count (4 bits)<br>

    <code>B:</code> type index (16 bits)<br>

    <code>C..G:</code> argument registers (4 bits each)

  </td>

  <td>Construct an array of the given type and size, filling it with the

    supplied contents. The type must be an array type. The array's

    contents must be single-word (that is,

    no arrays of <code>long</code> or <code>double</code>, but reference

    types are acceptable). The constructed

    instance is stored as a "result" in the same way that the method invocation

    instructions store their results, so the constructed instance must

    be moved to a register with an immediately subsequent

    <code>move-result-object</code> instruction (if it is to be used).</td>

</tr>

<tr>

  <td>25 3rc</td>

  <td>filled-new-array/range {vCCCC .. vNNNN}, type@BBBB</td>

  <td><code>A:</code> array size and argument word count (8 bits)<br>

    <code>B:</code> type index (16 bits)<br>

    <code>C:</code> first argument register (16 bits)<br>

    <code>N = A + C - 1</code></td>

  <td>Construct an array of the given type and size, filling it with

    the supplied contents. Clarifications and restrictions are the same

    as <code>filled-new-array</code>, described above.</td>

</tr>

<tr>

  <td>26 31t</td>

  <td>fill-array-data vAA, +BBBBBBBB <i>(with supplemental data as specified

    below in "<code>fill-array-data-payload</code> Format")</i></td>

  <td><code>A:</code> array reference (8 bits)<br>

    <code>B:</code> signed "branch" offset to table data pseudo-instruction

    (32 bits)

  </td>

  <td>Fill the given array with the indicated data. The reference must be

    to an array of primitives, and the data table must match it in type and

    must contain no more elements than will fit in the array. That is,

    the array may be larger than the table, and if so, only the initial

    elements of the array are set, leaving the remainder alone.

  </td>

</tr>

<tr>

  <td>27 11x</td>

  <td>throw vAA</td>

  <td><code>A:</code> exception-bearing register (8 bits)<br></td>

  <td>Throw the indicated exception.</td>

</tr>

<tr>

  <td>28 10t</td>

  <td>goto +AA</td>

  <td><code>A:</code> signed branch offset (8 bits)</td>

  <td>Unconditionally jump to the indicated instruction.

    <p class="note"><strong>Note:</strong>

    The branch offset must not be <code>0</code>. (A spin

    loop may be legally constructed either with <code>goto/32</code> or

    by including a <code>nop</code> as a target before the branch.)</p>

  </td>

</tr>

<tr>

  <td>29 20t</td>

  <td>goto/16 +AAAA</td>

  <td><code>A:</code> signed branch offset (16 bits)<br></td>

  <td>Unconditionally jump to the indicated instruction.

    <p class="note"><strong>Note:</strong>

    The branch offset must not be <code>0</code>. (A spin

    loop may be legally constructed either with <code>goto/32</code> or

    by including a <code>nop</code> as a target before the branch.)</p>

  </td>

</tr>

<tr>

  <td>2a 30t</td>

  <td>goto/32 +AAAAAAAA</td>

  <td><code>A:</code> signed branch offset (32 bits)<br></td>

  <td>Unconditionally jump to the indicated instruction.</td>

</tr>

<tr>

  <td>2b 31t</td>

  <td>packed-switch vAA, +BBBBBBBB <i>(with supplemental data as

    specified below in "<code>packed-switch-payload</code> Format")</i></td>

  <td><code>A:</code> register to test<br>

    <code>B:</code> signed "branch" offset to table data pseudo-instruction

    (32 bits)

  </td>

  <td>Jump to a new instruction based on the value in the

    given register, using a table of offsets corresponding to each value

    in a particular integral range, or fall through to the next

    instruction if there is no match.

  </td>

</tr>

<tr>

  <td>2c 31t</td>

  <td>sparse-switch vAA, +BBBBBBBB <i>(with supplemental data as

    specified below in "<code>sparse-switch-payload</code> Format")</i></td>

  <td><code>A:</code> register to test<br>

    <code>B:</code> signed "branch" offset to table data pseudo-instruction

    (32 bits)

  </td>

  <td>Jump to a new instruction based on the value in the given

    register, using an ordered table of value-offset pairs, or fall

    through to the next instruction if there is no match.

  </td>

</tr>

<tr>

  <td>2d..31 23x</td>

  <td>cmp<i>kind</i> vAA, vBB, vCC<br>

    2d: cmpl-float <i>(lt bias)</i><br>

    2e: cmpg-float <i>(gt bias)</i><br>

    2f: cmpl-double <i>(lt bias)</i><br>

    30: cmpg-double <i>(gt bias)</i><br>

    31: cmp-long

  </td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> first source register or pair<br>

    <code>C:</code> second source register or pair</td>

  <td>Perform the indicated floating point or <code>long</code> comparison,

    setting <code>a</code> to <code>0</code> if <code>b == c</code>,

    <code>1</code> if <code>b &gt; c</code>,

    or <code>-1</code> if <code>b &lt; c</code>.

    The "bias" listed for the floating point operations

    indicates how <code>NaN</code> comparisons are treated: "gt bias"

    instructions return <code>1</code> for <code>NaN</code> comparisons,

    and "lt bias" instructions return <code>-1</code>.

    <p>For example, to check to see if floating point

    <code>x &lt; y</code> it is advisable to use

    <code>cmpg-float</code>; a result of <code>-1</code> indicates that

    the test was true, and the other values indicate it was false either

    due to a valid comparison or because one of the values was

    <code>NaN</code>.</p>

  </td>

</tr>

<tr>

  <td>32..37 22t</td>

  <td>if-<i>test</i> vA, vB, +CCCC<br>

    32: if-eq<br>

    33: if-ne<br>

    34: if-lt<br>

    35: if-ge<br>

    36: if-gt<br>

    37: if-le<br>

  </td>

  <td><code>A:</code> first register to test (4 bits)<br>

    <code>B:</code> second register to test (4 bits)<br>

    <code>C:</code> signed branch offset (16 bits)</td>

  <td>Branch to the given destination if the given two registers' values

    compare as specified.

    <p class="note"><strong>Note:</strong>

    The branch offset must not be <code>0</code>. (A spin

    loop may be legally constructed either by branching around a

    backward <code>goto</code> or by including a <code>nop</code> as

    a target before the branch.)</p>

  </td>

</tr>

<tr>

  <td>38..3d 21t</td>

  <td>if-<i>test</i>z vAA, +BBBB<br>

    38: if-eqz<br>

    39: if-nez<br>

    3a: if-ltz<br>

    3b: if-gez<br>

    3c: if-gtz<br>

    3d: if-lez<br>

  </td>

  <td><code>A:</code> register to test (8 bits)<br>

    <code>B:</code> signed branch offset (16 bits)</td>

  <td>Branch to the given destination if the given register's value compares

    with 0 as specified.

    <p class="note"><strong>Note:</strong>

    The branch offset must not be <code>0</code>. (A spin

    loop may be legally constructed either by branching around a

    backward <code>goto</code> or by including a <code>nop</code> as

    a target before the branch.)</p>

  </td>

</tr>

<tr>

  <td>3e..43 10x</td>

  <td><i>(unused)</i></td>

  <td>&nbsp;</td>

  <td><i>(unused)</i></td>

</tr>

<tr>

  <td>44..51 23x</td>

  <td><i>arrayop</i> vAA, vBB, vCC<br>

    44: aget<br>

    45: aget-wide<br>

    46: aget-object<br>

    47: aget-boolean<br>

    48: aget-byte<br>

    49: aget-char<br>

    4a: aget-short<br>

    4b: aput<br>

    4c: aput-wide<br>

    4d: aput-object<br>

    4e: aput-boolean<br>

    4f: aput-byte<br>

    50: aput-char<br>

    51: aput-short

  </td>

  <td><code>A:</code> value register or pair; may be source or dest

      (8 bits)<br>

    <code>B:</code> array register (8 bits)<br>

    <code>C:</code> index register (8 bits)</td>

  <td>Perform the identified array operation at the identified index of

    the given array, loading or storing into the value register.</td>

</tr>

<tr>

  <td>52..5f 22c</td>

  <td>i<i>instanceop</i> vA, vB, field@CCCC<br>

    52: iget<br>

    53: iget-wide<br>

    54: iget-object<br>

    55: iget-boolean<br>

    56: iget-byte<br>

    57: iget-char<br>

    58: iget-short<br>

    59: iput<br>

    5a: iput-wide<br>

    5b: iput-object<br>

    5c: iput-boolean<br>

    5d: iput-byte<br>

    5e: iput-char<br>

    5f: iput-short

  </td>

  <td><code>A:</code> value register or pair; may be source or dest

      (4 bits)<br>

    <code>B:</code> object register (4 bits)<br>

    <code>C:</code> instance field reference index (16 bits)</td>

  <td>Perform the identified object instance field operation with

    the identified field, loading or storing into the value register.

    <p class="note"><strong>Note:</strong> These opcodes are reasonable candidates for static linking,

    altering the field argument to be a more direct offset.</p>

  </td>

</tr>

<tr>

  <td>60..6d 21c</td>

  <td>s<i>staticop</i> vAA, field@BBBB<br>

    60: sget<br>

    61: sget-wide<br>

    62: sget-object<br>

    63: sget-boolean<br>

    64: sget-byte<br>

    65: sget-char<br>

    66: sget-short<br>

    67: sput<br>

    68: sput-wide<br>

    69: sput-object<br>

    6a: sput-boolean<br>

    6b: sput-byte<br>

    6c: sput-char<br>

    6d: sput-short

  </td>

  <td><code>A:</code> value register or pair; may be source or dest

      (8 bits)<br>

    <code>B:</code> static field reference index (16 bits)</td>

  <td>Perform the identified object static field operation with the identified

    static field, loading or storing into the value register.

    <p class="note"><strong>Note:</strong> These opcodes are reasonable candidates for static linking,

    altering the field argument to be a more direct offset.</p>

  </td>

</tr>

<tr>

  <td>6e..72 35c</td>

  <td>invoke-<i>kind</i> {vC, vD, vE, vF, vG}, meth@BBBB<br>

    6e: invoke-virtual<br>

    6f: invoke-super<br>

    70: invoke-direct<br>

    71: invoke-static<br>

    72: invoke-interface

  </td>

  <td>

    <code>A:</code> argument word count (4 bits)<br>

    <code>B:</code> method reference index (16 bits)<br>

    <code>C..G:</code> argument registers (4 bits each)

  </td>

  <td>Call the indicated method. The result (if any) may be stored

    with an appropriate <code>move-result*</code> variant as the immediately

    subsequent instruction.

    <p><code>invoke-virtual</code> is used to invoke a normal virtual

    method (a method that is not <code>private</code>, <code>static</code>,

    or <code>final</code>, and is also not a constructor).</p>

    <p>When the <code>method_id</code> references a method of a non-interface

    class, <code>invoke-super</code> is used to invoke the closest superclass's

    virtual method (as opposed to the one with the same <code>method_id</code>

    in the calling class). The same method restrictions hold as for

    <code>invoke-virtual</code>.</p>

    <p>In Dex files version <code>037</code> or later, if the

    <code>method_id</code> refers to an interface method,

    <code>invoke-super</code> is used to invoke the most specific,

    non-overridden version of that method defined on that interface.  The same

    method restrictions hold as for <code>invoke-virtual</code>. In Dex files

    prior to version <code>037</code>, having an interface

    <code>method_id</code> is illegal and undefined.</p>

    <p><code>invoke-direct</code> is used to invoke a non-<code>static</code>

    direct method (that is, an instance method that is by its nature

    non-overridable, namely either a <code>private</code> instance method or a

    constructor).</p>

    <p><code>invoke-static</code> is used to invoke a <code>static</code>

    method (which is always considered a direct method).</p>

    <p><code>invoke-interface</code> is used to invoke an

    <code>interface</code> method, that is, on an object whose concrete

    class isn't known, using a <code>method_id</code> that refers to

    an <code>interface</code>.</p>

    <p class="note"><strong>Note:</strong> These opcodes are reasonable candidates for static linking,

    altering the method argument to be a more direct offset

    (or pair thereof).</p>

  </td>

</tr>

<tr>

  <td>73 10x</td>

  <td><i>(unused)</i></td>

  <td>&nbsp;</td>

  <td><i>(unused)</i></td>

</tr>

<tr>

  <td>74..78 3rc</td>

  <td>invoke-<i>kind</i>/range {vCCCC .. vNNNN}, meth@BBBB<br>

    74: invoke-virtual/range<br>

    75: invoke-super/range<br>

    76: invoke-direct/range<br>

    77: invoke-static/range<br>

    78: invoke-interface/range

  </td>

  <td><code>A:</code> argument word count (8 bits)<br>

    <code>B:</code> method reference index (16 bits)<br>

    <code>C:</code> first argument register (16 bits)<br>

    <code>N = A + C - 1</code></td>

  <td>Call the indicated method. See first <code>invoke-<i>kind</i></code>

    description above for details, caveats, and suggestions.

  </td>

</tr>

<tr>

  <td>79..7a 10x</td>

  <td><i>(unused)</i></td>

  <td>&nbsp;</td>

  <td><i>(unused)</i></td>

</tr>

<tr>

  <td>7b..8f 12x</td>

  <td><i>unop</i> vA, vB<br>

    7b: neg-int<br>

    7c: not-int<br>

    7d: neg-long<br>

    7e: not-long<br>

    7f: neg-float<br>

    80: neg-double<br>

    81: int-to-long<br>

    82: int-to-float<br>

    83: int-to-double<br>

    84: long-to-int<br>

    85: long-to-float<br>

    86: long-to-double<br>

    87: float-to-int<br>

    88: float-to-long<br>

    89: float-to-double<br>

    8a: double-to-int<br>

    8b: double-to-long<br>

    8c: double-to-float<br>

    8d: int-to-byte<br>

    8e: int-to-char<br>

    8f: int-to-short

  </td>

  <td><code>A:</code> destination register or pair (4 bits)<br>

    <code>B:</code> source register or pair (4 bits)</td>

  <td>Perform the identified unary operation on the source register,

    storing the result in the destination register.</td>

</tr>



<tr>

  <td>90..af 23x</td>

  <td><i>binop</i> vAA, vBB, vCC<br>

    90: add-int<br>

    91: sub-int<br>

    92: mul-int<br>

    93: div-int<br>

    94: rem-int<br>

    95: and-int<br>

    96: or-int<br>

    97: xor-int<br>

    98: shl-int<br>

    99: shr-int<br>

    9a: ushr-int<br>

    9b: add-long<br>

    9c: sub-long<br>

    9d: mul-long<br>

    9e: div-long<br>

    9f: rem-long<br>

    a0: and-long<br>

    a1: or-long<br>

    a2: xor-long<br>

    a3: shl-long<br>

    a4: shr-long<br>

    a5: ushr-long<br>

    a6: add-float<br>

    a7: sub-float<br>

    a8: mul-float<br>

    a9: div-float<br>

    aa: rem-float<br>

    ab: add-double<br>

    ac: sub-double<br>

    ad: mul-double<br>

    ae: div-double<br>

    af: rem-double

  </td>

  <td><code>A:</code> destination register or pair (8 bits)<br>

    <code>B:</code> first source register or pair (8 bits)<br>

    <code>C:</code> second source register or pair (8 bits)</td>

  <td>Perform the identified binary operation on the two source registers,

    storing the result in the destination register.

    <p class="note"><strong>Note:</strong>

    Contrary to other <code>-long</code> mathematical operations (which

    take register pairs for both their first and their second source),

    <code>shl-long</code>, <code>shr-long</code>, and <code>ushr-long</code>

    take a register pair for their first source (the value to be shifted),

    but a single register for their second source (the shifting distance).

    </p>

</td>

</tr>

<tr>

  <td>b0..cf 12x</td>

  <td><i>binop</i>/2addr vA, vB<br>

    b0: add-int/2addr<br>

    b1: sub-int/2addr<br>

    b2: mul-int/2addr<br>

    b3: div-int/2addr<br>

    b4: rem-int/2addr<br>

    b5: and-int/2addr<br>

    b6: or-int/2addr<br>

    b7: xor-int/2addr<br>

    b8: shl-int/2addr<br>

    b9: shr-int/2addr<br>

    ba: ushr-int/2addr<br>

    bb: add-long/2addr<br>

    bc: sub-long/2addr<br>

    bd: mul-long/2addr<br>

    be: div-long/2addr<br>

    bf: rem-long/2addr<br>

    c0: and-long/2addr<br>

    c1: or-long/2addr<br>

    c2: xor-long/2addr<br>

    c3: shl-long/2addr<br>

    c4: shr-long/2addr<br>

    c5: ushr-long/2addr<br>

    c6: add-float/2addr<br>

    c7: sub-float/2addr<br>

    c8: mul-float/2addr<br>

    c9: div-float/2addr<br>

    ca: rem-float/2addr<br>

    cb: add-double/2addr<br>

    cc: sub-double/2addr<br>

    cd: mul-double/2addr<br>

    ce: div-double/2addr<br>

    cf: rem-double/2addr

  </td>

  <td><code>A:</code> destination and first source register or pair

      (4 bits)<br>

    <code>B:</code> second source register or pair (4 bits)</td>

  <td>Perform the identified binary operation on the two source registers,

    storing the result in the first source register.

    <p class="note"><strong>Note:</strong>

    Contrary to other <code>-long/2addr</code> mathematical operations

    (which take register pairs for both their destination/first source and

    their second source), <code>shl-long/2addr</code>,

    <code>shr-long/2addr</code>, and <code>ushr-long/2addr</code> take a

    register pair for their destination/first source (the value to be

    shifted), but a single register for their second source (the shifting

    distance).

    </p>

  </td>

</tr>

<tr>

  <td>d0..d7 22s</td>

  <td><i>binop</i>/lit16 vA, vB, #+CCCC<br>

    d0: add-int/lit16<br>

    d1: rsub-int (reverse subtract)<br>

    d2: mul-int/lit16<br>

    d3: div-int/lit16<br>

    d4: rem-int/lit16<br>

    d5: and-int/lit16<br>

    d6: or-int/lit16<br>

    d7: xor-int/lit16

  </td>

  <td><code>A:</code> destination register (4 bits)<br>

    <code>B:</code> source register (4 bits)<br>

    <code>C:</code> signed int constant (16 bits)</td>

  <td>Perform the indicated binary op on the indicated register (first

    argument) and literal value (second argument), storing the result in

    the destination register.

    <p class="note"><strong>Note:</strong>

    <code>rsub-int</code> does not have a suffix since this version is the

    main opcode of its family. Also, see below for details on its semantics.

    </p>

  </td>

</tr>

<tr>

  <td>d8..e2 22b</td>

  <td><i>binop</i>/lit8 vAA, vBB, #+CC<br>

    d8: add-int/lit8<br>

    d9: rsub-int/lit8<br>

    da: mul-int/lit8<br>

    db: div-int/lit8<br>

    dc: rem-int/lit8<br>

    dd: and-int/lit8<br>

    de: or-int/lit8<br>

    df: xor-int/lit8<br>

    e0: shl-int/lit8<br>

    e1: shr-int/lit8<br>

    e2: ushr-int/lit8

  </td>

  <td><code>A:</code> destination register (8 bits)<br>

    <code>B:</code> source register (8 bits)<br>

    <code>C:</code> signed int constant (8 bits)</td>

  <td>Perform the indicated binary op on the indicated register (first

    argument) and literal value (second argument), storing the result

    in the destination register.

    <p class="note"><strong>Note:</strong> See below for details on the semantics of

    <code>rsub-int</code>.</p>

  </td>

</tr>

<tr>

  <td>e3..f9 10x</td>

  <td><i>(unused)</i></td>

  <td>&nbsp;</td>

  <td><i>(unused)</i></td>

</tr>

<tr>

  <td>fa 45cc</td>

  <td>invoke-polymorphic {vC, vD, vE, vF, vG}, meth@BBBB, proto@HHHH</td>

  <td>

    <code>A:</code> argument word count (4 bits) <br>

    <code>B:</code> method reference index (16 bits) <br>

    <code>C:</code> method handle reference to invoke (16 bits) <br>

    <code>D..G:</code> argument registers (4 bits each) <br>

    <code>H:</code> prototype reference index (16 bits) <br>

  </td>

  <td>

    Invoke the indicated method handle. The result (if any) may be stored

    with an appropriate <code>move-result*</code> variant as the immediately

    subsequent instruction.

    <p> The method reference must be to <code>java.lang.invoke.MethodHandle.invoke</code>

        or <code>java.lang.invoke.MethodHandle.invokeExact</code>.

    </p><p> The prototype reference describes the argument types provided

        and the expected return type.

    </p><p> The <code>invoke-polymorphic</code> bytecode may raise exceptions when it

        executes. The exceptions are described in the API documentation

        for <code>java.lang.invoke.MethodHandle.invoke</code> and

        <code>java.lang.invoke.MethodHandle.invokeExact</code>.

    </p><p> Present in Dex files from version <code>038</code> onwards.

  </p></td>

</tr>

<tr>

  <td>fb 4rcc</td>

  <td>invoke-polymorphic/range {vCCCC .. vNNNN}, meth@BBBB, proto@HHHH</td>

  <td>

    <code>A:</code> argument word count (8 bits) <br>

    <code>B:</code> method reference index (16 bits) <br>

    <code>C:</code> method handle reference to invoke (16 bits) <br>

    <code>H:</code> prototype reference index (16 bits) <br>

    <code>N = A + C - 1</code>

  </td>

  <td>

    Invoke the indicated method handle. See the <code>invoke-polymorphic</code>

    description above for details.

    <p> Present in Dex files from version <code>038</code> onwards.

  </p></td>

</tr>

<tr>

  <td>fc 35c</td>

  <td>invoke-custom {vC, vD, vE, vF, vG}, call_site@BBBB</td>

  <td>

    <code>A:</code> argument word count (4 bits) <br>

    <code>B:</code> call site reference index (16 bits) <br>

    <code>C..G:</code> argument registers (4 bits each)

  </td>

  <td> Resolves and invokes the indicated call site.

    The result from the invocation (if any) may be stored with an

    appropriate <code>move-result*</code> variant as the immediately

    subsequent instruction.



    <p> This instruction executes in two phases: call site

        resolution and call site invocation.



    </p><p> Call site resolution checks whether the indicated

      call site has an associated <code>java.lang.invoke.CallSite</code> instance.

      If not, the bootstrap linker method for the indicated call site is

      invoked using arguments present in the DEX file

      (see <a href="dex-format.html#call-site-item">call_site_item</a>). The

      bootstrap linker method returns

      a <code>java.lang.invoke.CallSite</code> instance that will then

      be associated with the indicated call site if no association

      exists. Another thread may have already made the association first,

      and if so execution of the instruction continues with the

      first associated <code>java.lang.invoke.CallSite</code> instance.



    </p><p> Call site invocation is made on the <code>java.lang.invoke.MethodHandle</code> target of the

      resolved <code>java.lang.invoke.CallSite</code> instance. The target is invoked as if

      executing <code>invoke-polymorphic</code> (described above)

      using the method handle and arguments to

      the <code>invoke-custom</code> instruction as the arguments to an

      exact method handle invocation.



    </p><p> Exceptions raised by the bootstrap linker method are wrapped

      in a <code>java.lang.BootstrapMethodError</code>.  A <code>BootstrapMethodError</code> is also raised if:

      </p><ul>

        <li>the bootstrap linker method fails to return a <code>java.lang.invoke.CallSite</code> instance.</li>

        <li>the returned <code>java.lang.invoke.CallSite</code> has a <code>null</code> method handle target.</li>

        <li>the method handle target is not of the requested type.</li>

      </ul>

    <p> Present in Dex files from version <code>038</code> onwards.

  </p></td>

</tr>

<tr>

  <td>fd 3rc</td>

  <td>invoke-custom/range {vCCCC .. vNNNN}, call_site@BBBB</td>

  <td>

    <code>A:</code> argument word count (8 bits) <br>

    <code>B:</code> call site reference index (16 bits) <br>

    <code>C:</code> first argument register (16-bits) <br>

    <code>N = A + C - 1</code>

  </td>

  <td>

    Resolve and invoke a call site. See the <code>invoke-custom</code> description above for details.

    <p> Present in Dex files from version <code>038</code> onwards.

  </p></td>

</tr>

<tr>

  <td>fe..ff 10x</td>

  <td><i>(unused)</i></td>

  <td>&nbsp;</td>

  <td><i>(unused)</i></td>

</tr>

</tbody>

</table>



## 压缩开关有效载荷格式

<hr / >

<table class="supplement">

<thead>

<tr>

  <th>Name</th>

  <th>Format</th>

  <th>Description</th>

</tr>

</thead>

<tbody>

<tr>

  <td>ident</td>

  <td>ushort = 0x0100</td>

  <td>identifying pseudo-opcode</td>

</tr>

<tr>

  <td>size</td>

  <td>ushort</td>

  <td>number of entries in the table</td>

</tr>

<tr>

  <td>first_key</td>

  <td>int</td>

  <td>first (and lowest) switch case value</td>

</tr>

<tr>

  <td>targets</td>

  <td>int[]</td>

  <td>list of <code>size</code> relative branch targets. The targets are

    relative to the address of the switch opcode, not of this table.

  </td>

</tr>

</tbody>

</table>



    ﻿注意：这个表中每一个实例的编码单元的总数为`(size * 2) + 4`。



## 稀疏开关有效载荷格式

<hr / >

<table class="supplement">

<thead>

<tr>

  <th>Name</th>

  <th>Format</th>

  <th>Description</th>

</tr>

</thead>

<tbody>

<tr>

  <td>ident</td>

  <td>ushort = 0x0200</td>

  <td>identifying pseudo-opcode</td>

</tr>

<tr>

  <td>size</td>

  <td>ushort</td>

  <td>number of entries in the table</td>

</tr>

<tr>

  <td>keys</td>

  <td>int[]</td>

  <td>list of <code>size</code> key values, sorted low-to-high</td>

</tr>

<tr>

  <td>targets</td>

  <td>int[]</td>

  <td>list of <code>size</code> relative branch targets, each corresponding

    to the key value at the same index. The targets are

    relative to the address of the switch opcode, not of this table.

  </td>

</tr>

</tbody>

</table>

    ﻿注意：这个表中每一个实例的编码单元的总数为`(size * 4) + 2`。



## 填充数组数据有效载荷格式

<hr / >

<table class="supplement">

<thead>

<tr>

  <th>Name</th>

  <th>Format</th>

  <th>Description</th>

</tr>

</thead>

<tbody>

<tr>

  <td>ident</td>

  <td>ushort = 0x0300</td>

  <td>identifying pseudo-opcode</td>

</tr>

<tr>

  <td>element_width</td>

  <td>ushort</td>

  <td>number of bytes in each element</td>

</tr>

<tr>

  <td>size</td>

  <td>uint</td>

  <td>number of elements in the table</td>

</tr>

<tr>

  <td>data</td>

  <td>ubyte[]</td>

  <td>data values</td>

</tr>

</tbody>

</table>

    ﻿注意：这个表中每一个实例的编码单元的总数为`(size * element_width + 1)  / 2 + 4`。



## 数学操作细节

<hr / >

    ﻿注意：浮点操作必须要遵守IEEE 754规则，使用舍入到最接近的和渐进下溢算法，除非在哪里明确指出。



<table class="math">

<thead>

<tr>

  <th>Opcode</th>

  <th>C Semantics</th>

  <th>Notes</th>

</tr>

</thead>

<tbody>

<tr>

  <td>neg-int</td>

  <td>int32 a;<br>

    int32 result = -a;

  </td>

  <td>Unary twos-complement.</td>

</tr>

<tr>

  <td>not-int</td>

  <td>int32 a;<br>

    int32 result = ~a;

  </td>

  <td>Unary ones-complement.</td>

</tr>

<tr>

  <td>neg-long</td>

  <td>int64 a;<br>

    int64 result = -a;

  </td>

  <td>Unary twos-complement.</td>

</tr>

<tr>

  <td>not-long</td>

  <td>int64 a;<br>

    int64 result = ~a;

  </td>

  <td>Unary ones-complement.</td>

</tr>

<tr>

  <td>neg-float</td>

  <td>float a;<br>

    float result = -a;

  </td>

  <td>Floating point negation.</td>

</tr>

<tr>

  <td>neg-double</td>

  <td>double a;<br>

    double result = -a;

  </td>

  <td>Floating point negation.</td>

</tr>

<tr>

  <td>int-to-long</td>

  <td>int32 a;<br>

    int64 result = (int64) a;

  </td>

  <td>Sign extension of <code>int32</code> into <code>int64</code>.</td>

</tr>

<tr>

  <td>int-to-float</td>

  <td>int32 a;<br>

    float result = (float) a;

  </td>

  <td>Conversion of <code>int32</code> to <code>float</code>, using

    round-to-nearest. This loses precision for some values.

  </td>

</tr>

<tr>

  <td>int-to-double</td>

  <td>int32 a;<br>

    double result = (double) a;

  </td>

  <td>Conversion of <code>int32</code> to <code>double</code>.</td>

</tr>

<tr>

  <td>long-to-int</td>

  <td>int64 a;<br>

    int32 result = (int32) a;

  </td>

  <td>Truncation of <code>int64</code> into <code>int32</code>.</td>

</tr>

<tr>

  <td>long-to-float</td>

  <td>int64 a;<br>

    float result = (float) a;

  </td>

  <td>Conversion of <code>int64</code> to <code>float</code>, using

    round-to-nearest. This loses precision for some values.

  </td>

</tr>

<tr>

  <td>long-to-double</td>

  <td>int64 a;<br>

    double result = (double) a;

  </td>

  <td>Conversion of <code>int64</code> to <code>double</code>, using

    round-to-nearest. This loses precision for some values.

  </td>

</tr>

<tr>

  <td>float-to-int</td>

  <td>float a;<br>

    int32 result = (int32) a;

  </td>

  <td>Conversion of <code>float</code> to <code>int32</code>, using

    round-toward-zero. <code>NaN</code> and <code>-0.0</code> (negative zero)

    convert to the integer <code>0</code>. Infinities and values with

    too large a magnitude to be represented get converted to either

    <code>0x7fffffff</code> or <code>-0x80000000</code> depending on sign.

  </td>

</tr>

<tr>

  <td>float-to-long</td>

  <td>float a;<br>

    int64 result = (int64) a;

  </td>

  <td>Conversion of <code>float</code> to <code>int64</code>, using

    round-toward-zero. The same special case rules as for

    <code>float-to-int</code> apply here, except that out-of-range values

    get converted to either <code>0x7fffffffffffffff</code> or

    <code>-0x8000000000000000</code> depending on sign.

  </td>

</tr>

<tr>

  <td>float-to-double</td>

  <td>float a;<br>

    double result = (double) a;

  </td>

  <td>Conversion of <code>float</code> to <code>double</code>, preserving

    the value exactly.

  </td>

</tr>

<tr>

  <td>double-to-int</td>

  <td>double a;<br>

    int32 result = (int32) a;

  </td>

  <td>Conversion of <code>double</code> to <code>int32</code>, using

    round-toward-zero. The same special case rules as for

    <code>float-to-int</code> apply here.

  </td>

</tr>

<tr>

  <td>double-to-long</td>

  <td>double a;<br>

    int64 result = (int64) a;

  </td>

  <td>Conversion of <code>double</code> to <code>int64</code>, using

    round-toward-zero. The same special case rules as for

    <code>float-to-long</code> apply here.

  </td>

</tr>

<tr>

  <td>double-to-float</td>

  <td>double a;<br>

    float result = (float) a;

  </td>

  <td>Conversion of <code>double</code> to <code>float</code>, using

    round-to-nearest. This loses precision for some values.

  </td>

</tr>

<tr>

  <td>int-to-byte</td>

  <td>int32 a;<br>

    int32 result = (a &lt;&lt; 24) &gt;&gt; 24;

  </td>

  <td>Truncation of <code>int32</code> to <code>int8</code>, sign

    extending the result.

  </td>

</tr>

<tr>

  <td>int-to-char</td>

  <td>int32 a;<br>

    int32 result = a &amp; 0xffff;

  </td>

  <td>Truncation of <code>int32</code> to <code>uint16</code>, without

    sign extension.

  </td>

</tr>

<tr>

  <td>int-to-short</td>

  <td>int32 a;<br>

    int32 result = (a &lt;&lt; 16) &gt;&gt; 16;

  </td>

  <td>Truncation of <code>int32</code> to <code>int16</code>, sign

    extending the result.

  </td>

</tr>

<tr>

  <td>add-int</td>

  <td>int32 a, b;<br>

    int32 result = a + b;

  </td>

  <td>Twos-complement addition.</td>

</tr>

<tr>

  <td>sub-int</td>

  <td>int32 a, b;<br>

    int32 result = a - b;

  </td>

  <td>Twos-complement subtraction.</td>

</tr>

<tr>

  <td>rsub-int</td>

  <td>int32 a, b;<br>

    int32 result = b - a;

  </td>

  <td>Twos-complement reverse subtraction.</td>

</tr>

<tr>

  <td>mul-int</td>

  <td>int32 a, b;<br>

    int32 result = a * b;

  </td>

  <td>Twos-complement multiplication.</td>

</tr>

<tr>

  <td>div-int</td>

  <td>int32 a, b;<br>

    int32 result = a / b;

  </td>

  <td>Twos-complement division, rounded towards zero (that is, truncated to

    integer). This throws <code>ArithmeticException</code> if

    <code>b == 0</code>.

  </td>

</tr>

<tr>

  <td>rem-int</td>

  <td>int32 a, b;<br>

    int32 result = a % b;

  </td>

  <td>Twos-complement remainder after division. The sign of the result

    is the same as that of <code>a</code>, and it is more precisely

    defined as <code>result == a - (a / b) * b</code>. This throws

    <code>ArithmeticException</code> if <code>b == 0</code>.

  </td>

</tr>

<tr>

  <td>and-int</td>

  <td>int32 a, b;<br>

    int32 result = a &amp; b;

  </td>

  <td>Bitwise AND.</td>

</tr>

<tr>

  <td>or-int</td>

  <td>int32 a, b;<br>

    int32 result = a | b;

  </td>

  <td>Bitwise OR.</td>

</tr>

<tr>

  <td>xor-int</td>

  <td>int32 a, b;<br>

    int32 result = a ^ b;

  </td>

  <td>Bitwise XOR.</td>

</tr>

<tr>

  <td>shl-int</td>

  <td>int32 a, b;<br>

    int32 result = a &lt;&lt; (b &amp; 0x1f);

  </td>

  <td>Bitwise shift left (with masked argument).</td>

</tr>

<tr>

  <td>shr-int</td>

  <td>int32 a, b;<br>

    int32 result = a &gt;&gt; (b &amp; 0x1f);

  </td>

  <td>Bitwise signed shift right (with masked argument).</td>

</tr>

<tr>

  <td>ushr-int</td>

  <td>uint32 a, b;<br>

    int32 result = a &gt;&gt; (b &amp; 0x1f);

  </td>

  <td>Bitwise unsigned shift right (with masked argument).</td>

</tr>

<tr>

  <td>add-long</td>

  <td>int64 a, b;<br>

    int64 result = a + b;

  </td>

  <td>Twos-complement addition.</td>

</tr>

<tr>

  <td>sub-long</td>

  <td>int64 a, b;<br>

    int64 result = a - b;

  </td>

  <td>Twos-complement subtraction.</td>

</tr>

<tr>

  <td>mul-long</td>

  <td>int64 a, b;<br>

    int64 result = a * b;

  </td>

  <td>Twos-complement multiplication.</td>

</tr>

<tr>

  <td>div-long</td>

  <td>int64 a, b;<br>

    int64 result = a / b;

  </td>

  <td>Twos-complement division, rounded towards zero (that is, truncated to

    integer). This throws <code>ArithmeticException</code> if

    <code>b == 0</code>.

  </td>

</tr>

<tr>

  <td>rem-long</td>

  <td>int64 a, b;<br>

    int64 result = a % b;

  </td>

  <td>Twos-complement remainder after division. The sign of the result

    is the same as that of <code>a</code>, and it is more precisely

    defined as <code>result == a - (a / b) * b</code>. This throws

    <code>ArithmeticException</code> if <code>b == 0</code>.

  </td>

</tr>

<tr>

  <td>and-long</td>

  <td>int64 a, b;<br>

    int64 result = a &amp; b;

  </td>

  <td>Bitwise AND.</td>

</tr>

<tr>

  <td>or-long</td>

  <td>int64 a, b;<br>

    int64 result = a | b;

  </td>

  <td>Bitwise OR.</td>

</tr>

<tr>

  <td>xor-long</td>

  <td>int64 a, b;<br>

    int64 result = a ^ b;

  </td>

  <td>Bitwise XOR.</td>

</tr>

<tr>

  <td>shl-long</td>

  <td>int64 a;<br>

    int32 b;<br>

    int64 result = a &lt;&lt; (b &amp; 0x3f);

  </td>

  <td>Bitwise shift left (with masked argument).</td>

</tr>

<tr>

  <td>shr-long</td>

  <td>int64 a;<br>

    int32 b;<br>

    int64 result = a &gt;&gt; (b &amp; 0x3f);

  </td>

  <td>Bitwise signed shift right (with masked argument).</td>

</tr>

<tr>

  <td>ushr-long</td>

  <td>uint64 a;<br>

    int32 b;<br>

    int64 result = a &gt;&gt; (b &amp; 0x3f);

  </td>

  <td>Bitwise unsigned shift right (with masked argument).</td>

</tr>

<tr>

  <td>add-float</td>

  <td>float a, b;<br>

    float result = a + b;

  </td>

  <td>Floating point addition.</td>

</tr>

<tr>

  <td>sub-float</td>

  <td>float a, b;<br>

    float result = a - b;

  </td>

  <td>Floating point subtraction.</td>

</tr>

<tr>

  <td>mul-float</td>

  <td>float a, b;<br>

    float result = a * b;

  </td>

  <td>Floating point multiplication.</td>

</tr>

<tr>

  <td>div-float</td>

  <td>float a, b;<br>

    float result = a / b;

  </td>

  <td>Floating point division.</td>

</tr>

<tr>

  <td>rem-float</td>

  <td>float a, b;<br>

    float result = a % b;

  </td>

  <td>Floating point remainder after division. This function is different

    than IEEE 754 remainder and is defined as

    <code>result == a - roundTowardZero(a / b) * b</code>.

  </td>

</tr>

<tr>

  <td>add-double</td>

  <td>double a, b;<br>

    double result = a + b;

  </td>

  <td>Floating point addition.</td>

</tr>

<tr>

  <td>sub-double</td>

  <td>double a, b;<br>

    double result = a - b;

  </td>

  <td>Floating point subtraction.</td>

</tr>

<tr>

  <td>mul-double</td>

  <td>double a, b;<br>

    double result = a * b;

  </td>

  <td>Floating point multiplication.</td>

</tr>

<tr>

  <td>div-double</td>

  <td>double a, b;<br>

    double result = a / b;

  </td>

  <td>Floating point division.</td>

</tr>

<tr>

  <td>rem-double</td>

  <td>double a, b;<br>

    double result = a % b;

  </td>

  <td>Floating point remainder after division. This function is different

    than IEEE 754 remainder and is defined as

    <code>result == a - roundTowardZero(a / b) * b</code>.

  </td>

</tr>

</tbody>

</table>
