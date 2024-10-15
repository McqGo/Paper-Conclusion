# T I R A M I S U: A Polyhedral Compiler for Expressing Fast and Portable Code

# T I R A M I S U: 一款用于表达快速且可移植代码的多面体编译器

---

## 核心思想

TIRAMISU的核心是四层IR+多面体。四层IR确保层级分离，而层级分离定义了应用优化的特定顺序，并确保给定层中的编译器阶段无需担心修改或撤销早期层做出的决策。

---

## 主要工作

本文提出TIRAMISU，一个多面体编译器框架，它具有一个调度语言，该语言包含针对多核 CPU、GPU 和分布式系统的命令。编译器使用四层中间表示来实现，该表示将算法、计算发生的时间和地点、数据布局和通信分开。

---

## 学到的知识点

1. TIRAMISU：一个多面体框架，旨在为包括多核、GPU 和分布式机器在内的多个平台生成高性能代码。引入了一种调度语言，其中包含新颖的命令，以明确管理针对这些系统时出现的复杂性。该框架专为图像处理、模板、线性代数和深度学习领域而设计。使用四级中间表示，允许算法、循环变换、数据布局和通信完全分离。这种分离简化了使用相同算法针对多个硬件架构的目标。已证明 TIRAMISU 在不同的硬件架构（包括多核 CPU、GPU 和分布式机器）上，其性能与现有编译器和库相当或优于最先进的编译器和手工调优的库。它具有一个调度语言，该语言具有用于控制数据通信、同步和映射到不同内存层次结构的新命令。这些扩展使目标能够针对多种高性能架构，包括多核 CPU、GPU 和分布式机器。TIRAMISU 是一种嵌入 C++ 的领域特定语言 (DSL)。它提供了一个 C++ API，允许用户编写高层次、与架构无关的算法，以及一组指导代码生成的调度命令。输入的 TIRAMISU 代码可以由程序员直接编写，也可以由其他 DSL 编译器生成。TIRAMISU 构建了一个高级中间表示 (IR)，应用用户指定的循环和数据布局转换，并生成利用目标硬件特性的优化后端代码（多核和分布式机器的 LLVM IR 以及 GPU 的 LLVM IR + CUDA）。

2. TIRAMISU 具有两个主要特点：它依赖于基于多面体模型的灵活表示，并拥有丰富的调度语言，允许对优化进行细粒度控制。TIRAMISU 使用四级中间表示，允许算法、循环变换、数据布局和通信完全分离。

3. **TIRAMISU的作用目标**：适合实现数据并行算法（操作数组的循环嵌套）。旨在表达数据并行算法，特别是那些使用循环嵌套和语句序列操作密集数组的算法。这些算法通常出现在图像处理、深度学习、稠密线性代数、张量运算和模板计算等领域。

4. TIRAMISU的分离思想：为了简化调度语言的实现，TIRAMISU明确地将中间表示分为四个层，旨在通过将与架构无关的算法与代码转换、数据布局和通信分离来隐藏执行平台的复杂性和多样性。

5. TIRAMISU的四层IR：四层 IR 将算法与代码转换和数据布局转换分离，从而实现可移植性并简化特定于架构的降低转换的组合。

6. **TIRAMISU比PolyMage这些全自动多面体编译器优秀的原因**：首先，这些编译器没有实现一些关键的优化，例如数组打包、寄存器阻塞、数据预取和异步通信（这些都是 TIRAMISU 支持的）；其次，它们没有精确的成本模型来决定哪些优化是有利可图的。例如，Pluto自动调度算法（用于 Pluto、PENCIL 和 Polly）试图最小化生产者和消费者语句之间的距离，同时最大化最外层并行性，但它没有考虑数据布局、冗余计算或生成的代码控制的复杂性。TIRAMISU 并不依赖于全自动调度，而是依赖于一组调度命令，让用户完全控制调度。

7. **TIRAMISU 并非完全自动**，而是依赖用户提供调度命令来控制生成代码中的选择（同步/异步通信、通信粒度、缓冲区大小、何时发送和接收、通信成本与重新计算成本等）。

8. **TIRAMISU比AlphaZ这些带有调度语言的多面体编译器优秀的原因**：由于这些框架是多面体的，因此它们可以表达任何仿射变换。然而，它们的调度语言并不针对分布式架构。相比之下，TIRAMISU 提供了用于分区计算（针对分布式系统）、同步和跨节点数据分布的调度命令。

9. **TIRAMISU比Halide这些带有调度语言的非多面体编译器优秀的原因**：Halide是一种图像处理 DSL，它使用调度语言，用区间表示迭代空间，而不是多面体模型。这限制了 Halide 的表达能力。例如，与 TIRAMISU 不同，Halide 无法自然地表示非矩形迭代空间，这就是为什么分布式 Halide在生成时会高估数据通信量（发送和接收）的原因。Halide 没有依赖分析，而TIRAMISU有。由于 TVM 也是一个非多面体编译器，因此 Halide 和 TIRAMISU 之间由于使用多面体模型而产生的差异也适用于 TVM。

10. TIRAMISU的算法指定：TIRAMISU 程序的第一部分指定了算法，但没有指定循环优化（何时何地进行计算）、数据布局（数据如何在内存中存储）或通信。在这个层面上，没有数据位置的概念；相反，值通过显式的生产者-消费者关系进行通信。

11. TIRAMISU的四种高级调度命令：
    
    - 循环嵌套变换命令：Commands for loop nest transformations，这些命令包括常见的仿射变换，例如循环平铺、拆分、移位等。
    
    - 将循环级别映射到硬件的命令：Commands for mapping loop levels to hardware，例如循环并行化、矢量化以及将循环级别映射到 GPU 块或线程维度。
    
    - 数据操作命令：Commands for manipulating data，包括 (1) 分配数组；(2) 设置数组属性，包括数组存储在主机、设备、共享或本地内存（GPU）中；(3) 复制数据（在内存层次结构之间或节点之间）；以及 (4) 设置数组访问。在大多数情况下，用户只需要使用高级数据操作命令。如果高级命令不够表达，用户可以使用更具表达力的低级命令。
    
    -  添加同步操作的命令：Commands for adding synchronization operations，用户可以声明一个屏障，或者使用发送和接收函数进行点对点同步。

12. **TIRAMISU使用四层IR的原因**：TIRAMISU 多层中间表示的主要目标是通过以特定顺序应用调度命令来简化调度命令的实现。大多数当前的中间表示不适合 TIRAMISU，因为大多数当前的中间表示使用内存来在程序语句之间进行通信。这在程序中创建了基于内存的依赖关系，并迫使编译器在决定优化和映射到硬件之前选择数据布局。由于基于内存的依赖关系限制了优化，因此针对不同硬件架构优化程序通常需要修改数据布局并消除基于内存的依赖关系 [31]。因此，在调度之前指定的任何数据布局都必须撤销，以允许更自由的调度，并且代码必须适应使用最适合目标硬件的数据布局。应用这些数据布局转换和消除基于内存的依赖关系具有挑战性。TIRAMISU 通过使用多层 IR 来解决代码生成中的这些复杂性，该 IR 将与架构无关的算法与循环变换、数据布局和通信完全分离。第一层表示使用生产者-消费者关系描述纯算法，不涉及内存位置。第二层指定计算顺序以及哪个处理器计算每个值；这一层适用于执行大量优化，而无需处理具体的内存布局。第三层指定在中间数据被使用前存储它们的位置。第四层添加所有必要的通信和同步操作。**层级分离定义了应用优化的特定顺序，并确保给定层中的编译器阶段无需担心修改或撤销早期层做出的决策**。例如，指定计算顺序和位置的阶段可以安全地假设不需要数据布局转换。这个简单的假设使 TIRAMISU 避免了依赖大量专注于数据布局转换以允许调度的研究。

13. **TIRAMISU的四层IR**：
    
    - 第一层（抽象算法）：TIRAMISU 的第一层指定了算法，但没有指定计算发生的时间和地点、数据如何在内存中存储或通信方式。值通过显式的生产者-消费者关系进行通信。
    
    - 第二层（计算管理）：TIRAMISU 的第二层指定了计算执行的顺序以及执行它们的处理器。这一层没有指定中间值如何在内存中存储；这简化了优化过程，因为这些转换不需要执行复杂的数据布局转换。第一层到第二层的转换是使用调度命令自动完成的。本层计算按顺序分配给特定处理器；顺序由时间维度和空间维度决定。时间维度指定相对于其他计算的执行顺序，而空间维度指定每个计算在哪个处理器上执行。空间维度使用标签与时间维度区分，标签由处理器类型组成。
    
    -  第三层（数据管理）：第三层通过指定中间值的存储位置来具体化数据布局。在这个层级中，还会构建任何必要的缓冲区分配/释放。TIRAMISU 通过应用数据映射的调度命令，自动从第二层生成该层。数据管理层指定用于存储计算值的内存位置。它包含 II 层表示以及分配/释放语句，以及一组访问关系，这些关系将 II 层的计算映射到该计算读取或写入的数组元素。标量被视为单元素数组。对于每个缓冲区，都会创建一个分配语句，指定缓冲区的类型和大小。类似地，还会添加一个释放语句。
    
    - 第四层（通信管理）：第四层将同步和通信操作添加到表示中，将它们映射到时空域，并具体化。当发生缓冲区分配/释放语句时。该层由用户指定的命令从第三层自动生成。在第三层中添加的任何分配或释放操作也会映射到该层的时空域。

14. **程序如何在TIRAMISU的四层IR间单向流动**：
    
    - 将第一层转换为第二层：将第一层转换为第二层使用两种类型的调度命令：(1) 循环嵌套变换命令（例如 tile()、split()、shift()、interchange()）；(2) 将循环层映射到硬件的命令（包括 parallelize()、vectorize()、gpu()）。
    
    - 将第二层转换为第三层：这是通过在第二层添加访问关系来实现的。默认情况下，TIRAMISU 使用身份访问关系（即，将计算 C(i,j) 存储到缓冲区 C[i,j] 中的访问关系）。如果使用 store_in() 命令，则访问关系将从该命令中推断出来。在将第二层转换为第三层时，还会添加缓冲区分配。调度命令 b.allocate_at(C, i) 创建一个新语句，该语句在计算 C 的相同循环嵌套中，但在循环级别 i 处分配缓冲区 b。
    
    - 将第三层转换为第四层：用于数据通信（发送和接收）、同步以及在全局、共享和本地内存之间复制数据的调度命令都将被转换为语句。例如，send() 和 receive() 命令将被转换为函数调用，这些函数调用将在代码生成期间被转换为 MPI 调用。

15. **TIRAMISU的代码生成**：从 Layer IV 中的一组计算生成代码相当于生成嵌套循环，这些循环访问集合中的每个计算，且仅访问一次，同时遵循计算之间的字典序排序。TIRAMISU 依赖于 ISL 库提供的 Cloog代码生成算法的实现。TIRAMISU 代码生成器接受 Layer IV IR 并生成抽象语法树 (AST)。然后遍历 AST 以生成针对特定硬件架构（取决于目标后端）的更低级别代码。
    
    - 多核 CPU 代码生成器从 AST 生成 LLVM IR。为了生成 LLVM IR，我们使用 Halide 作为库：我们首先生成 Halide IR，然后使用 Halide 将 Halide IR 降低到 LLVM IR。我们不使用 Halide 进行任何高级代码优化。所有代码优化都在生成 Halide IR 之前由 TIRAMISU 完成。然后，Halide 编译器将 Halide IR 循环降低到 LLVM IR。
    
    - GPU 代码生成器为主机代码生成 LLVM IR，为内核代码生成 CUDA。数据复制命令和关于缓冲区存储位置（共享、常量或全局内存）的信息都在第四层提供。TIRAMISU 将这些信息转换为生成的代码中的等效 CUDA 数据复制调用和缓冲区分配。带有 GPU 线程或 GPU 块标签的计算维度在降低的代码中被转换为相应的 GPU 线程和块 ID。TIRAMISU 代码生成器可以生成合并的数组访问，并可以使用共享内存和常量内存。它还可以通过将完整瓦片（循环嵌套的大小是瓦片大小的倍数）与部分瓦片（循环的剩余部分）分离来避免线程发散。
    
    - 针对分布式内存系统的代码生成器使用 MPI。在代码生成过程中，所有用于数据复制的函数调用都被转换为等效的 MPI 函数调用。生成的代码经过后处理，每个分布式循环都被转换为基于执行进程的 MPI 秩的条件语句。

---

## 个人思考

TIRAMISU的亮点就是四层IR，使用了关注点分离原则，每层解决特定的问题，而不过度耦合下一层的功能。这样一来，从效果上就可以尽可能从不同方面充分暴露源程序，提供更多的优化机会，从工程实现上，也更容易实现。结合我之前的认识，似乎是IR层数越多，就会使得优化的机会越多，编译器的优化力度和效果就会越大越好。



