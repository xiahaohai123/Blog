# SonarQube 介绍

## 什么是 SonarQube?

### 印象 SonarQube

直接来到官方网站，我们可以看到上面有来自官方的口号:

> Your teammate for Code Quality and Code Security

> SonarQube empowers all developers to write cleaner and safer code

> Enhance Your Workflow with Continuous Code Quality & Code Security

看完这三句口号，我们对 SonarQube 就会有一个大致的印象:

- **它是一个提升代码质量与代码安全的工具，并且可以以一种持续保证代码质量的方式集成进 devops 平台**

### SonarQube 如何工作？

来自官方文档的首页，有对 SonarQube 的大体介绍。

SonarQube 是一个自成体系的自动代码审查工具，它会帮助我们持续发布 Clean Code。SonarQube 可以分析三十多种编程语言，并且可以集成进 CI 管道以进行持续性的代码分析，以此确保我们的代码符合高质量标准。

## 编写 Clean Code

编写 Clean Code 对于维护健康的代码库至关重要。Sonar 将 Clean Code 定义为满足特定定义标准的代码，即可靠、安全、可维护、可读和模块化的代码，此外还具有其他关键属性。这适用于所有代码：源代码、测试代码、基础设施代码、胶水代码、脚本等。

而 Sonar 对该方法论的实现理论是 Clean as You Code。

## Clean as You Code

Clean as You Code 消除了人工审核代码所产生的许多缺陷。这个方法使用一个叫做 Quality Gate 的东西，当我们的**新代码**中有问题需要修复或检查时，Quality Gate 会提醒开发者，以此让开发者保持高标准的代码质量。

### 聚焦新代码

在该方法论中，我们大多数时候仅关注新代码，即哪些相较于旧版本出现新增或变更的代码。我们只要保证我们变更的代码是洁净的、安全的即可。

### 责任制度

一般情况下，我们不对过去老旧的代码以及其他人的代码负责，我们仅负责我们自己写的新代码。

### Quality Gate

中译质量门，SonarQube 的翻译插件会翻译为质量阈。

本质上是一个类似于关口的东西，可以用网关、海关等概念进行类比，Quality Gate 里面有一组判定条件，Quality Gate 通过这些判定条件来判断我们的项目是否达到了可发布水准。

在 Clean as You Code 方法论中，我们的 Quality Gate 应当保证两件事:

- 仅关注新代码
- 判定标准要高

这样，我们的工作便具有了**可持续性**，并且能保证持续产出**高质量、高安全性**的代码。

## 一些基础概念

仅仅知道 Quality Gate 是无法理解 SonarQube 的详细工作流程的。在这里我们举例一些 Quality Gate 用以当作判定标准的概念以帮助读者更清晰地理解这个方案的执行过程。

我们举例一些比较直观的概念与例子，例如我们可以为 Quality Gate 指定一个条件: Bugs 大于2，这意味着一旦本次分析后发现 Bugs 大于2时，Quality Gate 就会报错，认为有问题。

而这里的 Bug 指的是什么呢？我们这里就会举例一些在使用 SonarQube 中接触频率最高的概念:

- Bug: 表示代码中有错误，有可靠性问题。这个问题可能目前还没有在用户那边出现，但这并不意味着安全，迟早这个 Bug 会引发问题，最差结果就是它在最糟糕的时候引发了问题。例如在一个安全要求很高的生产环境上，凌晨3点因为这个 Bug 而出现了生产事故。
- Vulnerability (漏洞): 这意味着代码中存在后门，会引发安全问题。
- Security Hotspot (复审热点): 需要人工审核的存在安全敏感信息的代码片段。
- Code Smell (异味): 表示代码中存在一些与可维护性相关的问题，这个问题会导致维护人员在更改代码时会比正常情况下更加困难，付出更高的成本。最坏的情况下，维护人员会对代码感到困惑，并会在**更改时引入其他问题**。
- Cost/Remediation Cost (修复成本): 表示修复漏洞和可靠性问题所需要的时间。
- Debt/Technical Debt (技术债务): 表示修复可维护性问题所需要的时间成本。

## 参考文献

1. [sonarqube.org](https://www.sonarqube.org/)
2. [SonarQube Documentation](https://docs.sonarqube.org/latest/)
3. [Clean as You Code](https://docs.sonarqube.org/latest/user-guide/clean-as-you-code/)
4. [Concepts](https://docs.sonarqube.org/latest/user-guide/concepts/)