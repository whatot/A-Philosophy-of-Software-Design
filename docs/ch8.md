# 第八章 降低复杂性

本章介绍了有关如何创建更深层类的另一种思考方式。假设您正在开发一个新模块，并且发现了一个不可避免的复杂性。哪个更好：应该让模块用户处理复杂性，还是应该在模块内部处理复杂性？如果复杂度与模块提供的功能有关，则第二个答案通常是正确的答案。大多数模块拥有的用户多于开发人员，因此开发人员遭受的苦难要大于用户。作为模块开发人员，您应该努力使模块用户的生活尽可能轻松，即使这对您来说意味着额外的工作。表达此想法的另一种方法是，模块具有简单的接口比简单的实现更为重要。

作为开发人员，很容易以相反的方式行事：解决简单的问题，然后将困难的问题推给其他人。如果出现不确定如何处理的条件，最简单的方法是引发异常并让调用方处理它。如果不确定要实施什么策略，则可以定义一些配置参数来控制该策略，然后由系统管理员自行确定最佳策略。

这样的方法短期内会使您的生活更轻松，但它们会加剧复杂性，因此许多人必须处理一个问题，而不仅仅是一个人。例如，如果一个类抛出异常，则该类的每个调用者都必须处理该异常。如果一个类导出配置参数，则每个安装中的每个系统管理员都必须学习如何设置它们。

## 8.1 Example: editor text class 示例：编辑器文本类

考虑为 GUI 文本编辑器管理文件文本的类，这在第 6 章和第 7 章中讨论过。该类提供了将文件从磁盘读入内存、查询和修改文件在内存中的副本以及将修改后的版本写回磁盘的方法。当学生必须实现这个类时，他们中的许多人选择了一个面向行的接口，该接口具有读取、插入和删除整行文本的方法。这导致了类的简单实现，但也为更高级别的软件带来了复杂性。在用户界面级别，操作很少涉及整行。例如，击键会导致在现有行中插入单个字符;复制或删除选择项可以修改几个不同行的部分。使用面向行的文本界面，为了实现用户界面，高级软件必须分割和连接行。

面向字符的界面（如 6.3 节中所述）降低了复杂性。用户界面软件现在可以插入和删除任意范围的文本，而无需分割和合并行，因此变得更加简单。文本类的实现可能会变得更加复杂：如果内部将文本表示为行的集合，则必须拆分和合并行以实现面向字符的操作。这种方法更好，因为它封装了在文本类中拆分和合并的复杂性，从而降低了系统的整体复杂性。

## 8.2 Example: configuration parameters 示例：配置参数

配置参数是提高复杂度而不是降低复杂度的一个示例。类可以在内部输出一些控制其行为的参数，而不是在内部确定特定的行为，例如高速缓存的大小或在放弃之前重试请求的次数。然后，该类的用户必须为参数指定适当的值。在当今的系统中，配置参数已变得非常流行。有些系统有数百个。

拥护者认为配置参数不错，因为它们允许用户根据他们的特定要求和工作负载来调整系统。在某些情况下，低级基础结构代码很难知道要应用的最佳策略，而用户则对其域更加熟悉。例如，用户可能知道某些请求比其他请求更紧迫，因此用户为这些请求指定更高的优先级是有意义的。在这种情况下，配置参数可以在更广泛的域中带来更好的性能。

但是，配置参数还提供了一个轻松的借口，可以避免处理重要问题并将其传递给其他人。在许多情况下，用户或管理员很难或无法确定参数的正确值。在其他情况下，可以通过在系统实现中进行一些额外的工作来自动确定正确的值。考虑必须处理丢失数据包的网络协议。如果它发送请求但在一定时间内未收到响应，则重新发送该请求。确定重试间隔的一种方法是引入配置参数。但是，传输协议可以通过测量成功请求的响应时间，然后将其倍数用于重试间隔，自己计算出一个合理的值。这种方法降低了复杂性，使用户不必找出正确的重试间隔。它具有动态计算重试间隔的其他优点，因此，如果操作条件发生变化，它将自动进行调整。相反，配置参数很容易过时。

因此，您应尽可能避免使用配置参数。在导出配置参数之前，请问自己：“用户（或更高级别的模块）是否能够确定比我们在此确定的更好的值？” 当您创建配置参数时，请查看是否可以自动计算合理的默认值，因此用户仅需在特殊情况下提供值即可。理想情况下，每个模块都应完全解决问题。配置参数导致解决方案不完整，从而增加了系统复杂性。

## 8.3 Taking it too far 走得太远

降低复杂性时要谨慎处理；这个想法很容易被夸大。一种极端的方法是将整个应用程序的所有功能归为一个类，这显然没有意义。如果（a）被降低的复杂度与该类的现有功能密切相关，（b）降低复杂度将导致应用程序中其他地方的许多简化，则降低复杂度最有意义。简化了类的界面。请记住，目标是最大程度地降低整体系统复杂性。

第 6 章介绍了一些学生如何在文本类中定义反映用户界面的方法，例如实现退格键功能的方法。这似乎很好，因为它可以降低复杂性。但是，将用户界面的知识添加到文本类中并不会大大简化高层代码，并且用户界面的知识与文本类的核心功能无关。在这种情况下，降低复杂度只会导致信息泄漏。

## 8.4 Conclusion 结论

在开发模块时，请寻找机会减轻自己的痛苦，以减轻用户的痛苦。
