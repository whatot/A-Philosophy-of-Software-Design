# 第六章 通用模块更深入

设计新模块时，您将面临的最普遍的决定之一就是是以通用还是专用方式实现它。有人可能会争辩说，您应该采用通用方法，在这种方法中，您将实现一种可用于解决广泛问题的机制，而不仅是当今重要的问题。在这种情况下，新机制可能会在将来发现意外用途，从而节省时间。通用方法似乎与第 3 章中讨论的投资思路一致，在这里您花了更多时间在前面，以节省以后的时间。

另一方面，我们知道很难预测软件系统的未来需求，因此通用解决方案可能包含从未真正需要的功能。此外，如果您实现的东西过于通用，那么可能无法很好地解决您今天遇到的特定问题。结果，有些人可能会争辩说，最好只关注当今的需求，构建您所知道的需求，并针对您今天打算使用的方式进行专门化处理。如果您采用特殊用途的方法并在以后发现更多用途，则始终可以对其进行重构以使其通用。专用方法似乎与软件开发的增量方法一致。

## 6.1 使类变得通用

以我的经验，最有效的方法是以某种通用的方式实现新模块。短语“有点通用”表示该模块的功能应反映您当前的需求，但其接口则不应。相反，该接口应该足够通用以支持多种用途。该界面应易于使用，以满足当今的需求，而不必专门与它们联系在一起。“有点”这个词很重要：不要被带走并建造通用的东西，以致于很难满足当前的需求。

通用方法最重要的（也许是令人惊讶的）好处是，与专用方法相比，它导致更简单，更深入的界面。如果您将该类用于其他目的，则通用方法还可以节省将来的时间。但是，即使该模块仅用于其原始用途，由于其简单性，通用方法仍然更好。

## 6.2 示例：为编辑器存储文本

让我们考虑一个软件设计课程的示例，其中要求学生构建简单的 GUI 文本编辑器。编辑器必须显示一个文件，并允许用户指向，单击并键入以编辑该文件。编辑者必须在不同的窗口中支持同一文件的多个同时视图。他们还必须支持多级撤消和重做以修改文件。

每个学生项目都包括一个管理文件的基础文本的类。文本类通常提供以下方法：将文件加载到内存，读取和修改文件的文本以及将修改后的文本写回到文件。

许多学生团队为文本课实现了专用的 API。他们知道该类将在交互式编辑器中使用，因此他们考虑了编辑器必须提供的功能，并针对这些特定功能定制了文本类的 API。例如，如果编辑者的用户键入了退格键，则编辑者会立即删除光标左侧的字符；如果用户键入删除键，则编辑器立即删除光标右侧的字符。知道这一点后，一些团队在文本类中创建了一个方法来支持以下每个特定功能：

```java
void backspace(Cursor cursor);

void delete(Cursor cursor);
```

这些方法中的每一个都以光标位置作为参数。特殊类型的光标表示此位置。编辑器还必须支持可以复制或删除的选择。学生通过定义选择类并在删除过程中将该类的对象传递给文本类来解决此问题：

```java
void deleteSelection(Selection selection);
```

学生们可能认为，如果文本类的方法与用户可见的功能相对应，则将更易于实现用户界面。但是，实际上，这种专业化对用户界面代码几乎没有好处，并且为使用用户界面或文本类的开发人员带来了很高的认知负担。文本类以大量浅层方法结束，每种浅层方法仅适用于一个用户界面操作。许多方法（例如 delete）仅在单个位置调用。结果，在用户界面上工作的开发人员必须学习大量有关文本类的方法。

这种方法在用户界面和文本类之间造成了信息泄漏。与用户界面有关的抽象（例如选择或退格键）反映在文本类中；这增加了从事文本课的开发人员的认知负担。每个新的用户界面操作都需要在文本类中定义一个新方法，因此使用该用户界面的开发人员也可能最终也要使用该文本类。类设计的目标之一是允许每个类独立开发，但是专用方法将用户界面和文本类联系在一起。

## 6.3 更通用的 API

更好的方法是使文本类更通用。仅应根据基本文本功能定义其 API，而不应反映将用其实现的更高级别的操作。例如，只需两种方法即可修改文本：

```java
void insert(Position position, String newText);

void delete(Position start, Position end);
```

第一种方法在文本内的任意位置插入任意字符串，第二种方法删除大于或等于开始但小于结束的位置处的所有字符。此 API 还使用了更通用的 Position 类型来代替 Cursor，它反映了特定的用户界面。文本类还应该提供用于操纵文本中位置的通用工具，例如：

```java
Position changePosition(Position position, int numChars);
```

此方法返回一个新位置，该位置与给定位置相距给定字符数。如果 numChars 参数为正，则新位置在文件中比位置晚；如果 numChars 为负，则新位置在位置之前。必要时，该方法会自动跳到下一行或上一行。使用这些方法，可以使用以下代码来实现删除键（假定 cursor 变量保留当前光标的位置）：

```java
text.delete(cursor, text.changePosition(cursor, 1));
```

同样，可以按以下方式实现退格键：

```java
text.delete(text.changePosition(cursor, -1), cursor);
```

使用通用文本 API，实现用户界面功能（如删除和退格）的代码比使用专用文本 API 的原始方法要长一些。但是，新代码比旧代码更明显。在用户界面模块中工作的开发人员可能会关心由 Backspace 键删除哪些字符。使用新代码，这是显而易见的。使用旧代码，开发人员必须转到文本类并阅读退格方法的文档和/或代码以验证行为。此外，通用方法总体上比专用方法具有更少的代码，因为它用较少数量的通用方法代替了文本类中的大量专用方法。

使用通用接口实现的文本类除交互式编辑器外，还可以用于其他目的。作为一个示例，假设您正在构建一个应用程序，该应用程序通过将所有出现的特定字符串替换为另一个字符串来修改指定文件。专用文本类中的方法（例如，退格键和 Delete）对于此应用程序几乎没有价值。但是，通用文本类已经具有新应用程序所需的大多数功能。缺少的只是一种搜索给定字符串的下一个匹配项的方法，例如：

```java
Position findNext(Position start, String string);
```

当然，交互式文本编辑器可能具有搜索和替换的机制，在这种情况下，文本类将已经包含此方法。

## 6.4 通用性可以更好地隐藏信息

通用方法在文本和用户界面类之间提供了更清晰的分隔，从而可以更好地隐藏信息。文本类不需要知道用户界面的详细信息，例如如何处理退格键。这些细节现在封装在用户界面类中。可以添加新的用户界面功能，而无需在文本类中创建新的支持功能。通用界面还减轻了认知负担：使用用户界面的开发人员只需要学习一些简单的方法，就可以将其重复用于各种目的

文本类原始版本中的 backspace 方法是错误的抽象。它旨在隐藏有关删除哪些字符的信息，但是用户界面模块确实需要知道这一点。用户界面开发人员可能会阅读退格方法的代码，以确认其精确的行为。将方法放在文本类中只会使用户界面开发人员更难获得所需的信息。软件设计最重要的元素之一就是确定谁需要知道什么以及何时知道。当细节很重要时，最好使它们明确且尽可能明显，例如修订的 Backspace 操作实现。将这些信息隐藏在界面后面只会产生晦涩感。

## 6.5 问自己的问题

识别干净的通用类设计要比创建一个简单。您可以问自己一些问题，这将帮助您在接口的通用和专用之间找到适当的平衡。

满足我当前所有需求的最简单的界面是什么？如果减少 API 中的方法数量而不降低其整体功能，则可能正在创建更多通用的方法。专用文本 API 至少具有三种删除文本的方法：退格，删除和 deleteSelection。通用性更强的 API 只有一种删除文本的方法，可同时满足所有三个目的。仅在每种方法的 API 保持简单的前提下，减少方法的数量才有意义。如果您必须引入许多其他参数以减少方法数量，那么您可能并没有真正简化事情。

在多少情况下会使用此方法？如果一种方法是为特定用途而设计的，例如退格方法，那是一个危险信号，它可能太特殊了。看看是否可以用一个通用方法替换几种专用方法。

这个 API 是否易于使用以满足我当前的需求？这个问题可以帮助您确定何时使 API 变得简单而通用。如果您必须编写许多其他代码才能将类用于当前用途，那么这是一个危险信号，即该接口未提供正确的功能。例如，针对文本类的一种方法是围绕单字符操作进行设计：insert 插入单个字符，而 delete 删除单个字符。该 API 既简单又通用。但是，对于文本编辑器来说并不是特别容易使用：更高级别的代码将包含许多循环，用于插入或删除字符范围。单字符方法对于大型操作也将是低效的。

## 6.6 结论

通用接口比专用接口具有许多优点。它们往往更简单，使用的方法更少。它们还提供了类之间的更清晰的分隔，而专用接口则倾向于在类之间泄漏信息。使模块具有某种通用性是降低整体系统复杂性的最佳方法之一。