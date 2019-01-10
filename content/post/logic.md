---
title: "计算机数理逻辑"
date: 2019-01-10T09:09:59+08:00
draft: false
---

[TOC]

## Binary Decision & Graph Based Algorithm

### 二叉有向无环/左0右1

### Shannon deduction - two situation(same result,same 结构) - code

### 图和表达式相互转换

### 测试集



## *!The Quest for Efficient Boolean Satisfiability Solvers 

给定一个命题公式，确定是否存在变量赋值使得公式求值为true称为布尔可满足性问题，通常缩写为SAT。

这项研究已经导致了几种SAT算法的开发看到了实际成功。 其中一些算法是 **完整的** ，而其他算法是 **随机**方法。

近年来基于搜索的算法基于着名的Davis-Logemann-Loveland算法(因为历史原因有时称为DPLL算法）正在成为一些最有效的SAT求解器。

基于DPLL的SAT求解器是一个相对较小的软件。 许多求解器只有几千行代码（这些求解器是出于效率原因，主要用C或C ++编写。 但是，涉及的算法非常复杂，很多注意力都集中在求解器的各个方面例如编码，数据结构，选择算法和启发式，以及参数调整。 尽管整体框架已经得到了很好的理解，但人们已经有了在这方面工作多年，似乎我们已经达到了什么样的稳定水平可以在实践中实现 - 但是我们觉得许多开放性问题仍然存在，这提供了许多研究机会。

### 基本的DPLL框架&代码

为了求解器的效率，通常使用命题公式实例在Sum形式的产品中呈现，通常称为 **Conjunctive Normal Form**

（ **CNF** ）。将任何命题公式转换为一个CNF公式具有与原命题相同的可满足性。

CNF中一个SAT实例是 **一个或多个子句** 的 逻辑 **和** ，其中每个子句是 **一个或更多的 字面量** 的逻辑**或** 。 字面量是一个正的或负的**变量** 。

**如果公式中存在一个包含所有子句的字面量为0，然后是当前变量赋值或任何变量包含此内容的赋值将无法满足公式。 有一个子句分配给值0的所有字面量称为 冲突子句 。？**

最初，没有对变量赋值。 我们将未分配的变量称为 **自由**变量。 开始求解器将对要解决的实例进行一些预处理，由函数preprocess()。如果预处理无法确定结果，则主要循环自由变量上的分支开始，为其赋值。 我们称之为操作对变量的 **决策** ，变量将具有**决策级别**与之相关联，从1开始并随后的决定递增。这是由函数decision_next_branch()完成。在分支之后，由于这一决定及其后果，问题得到了简化。 功能deduce()执行一些推理来确定变量赋值在目前的一系列决策下，问题是可以满足的。变量在分支机构承担后，由此扣除分配的结果与决策变量相同的决策级别。 扣除后，如果所有条款都是**满足**，那么实例是可以满足的; 如果存在冲突子句，那么选择的当前分支不能导致令人满意的赋值，因此求解器将会**回溯**。要回溯到哪个决策级别由analyze_conflict()函数确定。回溯到0级表示即使没有任何分支，实例仍然是不可满足的。在那种情况下，求解器将声明该实例不可满足。 在功能内analyze_conflict()，解算器可以做一些分析并记录一些来自当前冲突的信息，以便为未来修剪搜索空间。这个过程称为 **冲突驱动的学习** 。如果实例既不满足也不满足在当前变量赋值下冲突，求解器将选择另一个变量分支并重复该过程。

DPLL递归描述：

```c++
DPLL(formula, assignment) {
	necessary = deduction(formula, assignment);
	new_asgnmnt = union(necessary, assignment);
	if (is_satisfied(formula, new_asgnmnt))
		return SATISFIABLE;
	else if (is_conflicting(formula, new_asgnmnt))
		return CONFLICT;
	var = choose_free_variable(formula, new_asgnmnt);
	asgn1 = union(new_asgnmnt, assign(var, 1));
	if (DPLL(formula, asgn1)==SATISFIABLE)
		return SATISFIABLE;
	else {
		asgn2 = union (new_asgnmnt, assign(var, 0));
		return DPLL(formula, asgn2);
	}
}
```

DPLL迭代描述：

```c++
status = preprocess();
if (status!=UNKNOWN) return status;
while(1) {
	decide_next_branch();
	while (true) {
		status = deduce();
		if (status == CONFLICT) {
			blevel = analyze_conflict(); // conflict-driven learning
			if (blevel == 0)
				return UNSATISFIABLE;
			else backtrack(blevel);
		}
		else if (status == SATISFIABLE)
			return SATISFIABLE;
		else break;
	}
}
```

- decide_next_brach():calculating counts for branching

#### 启发式分支 The Branching Heuristics

- 分支发生在函数decision_next_branch()，从所有自由变量中选择一个赋值。选择良好的分支严重影响的算法的效率。
- 可变状态的启发式算法独立衰减总和（VSIDS），保持每个阶段一个分数。VSIDS每当一个添加的子句包含变量时提高分数变量。 而且，作为搜索过程中，周期性地将所有分数除以常数。 事实上，VSIDS得分是一个文字最近出现次数越多，权重越高。 VSIDS选择分数最高的自由变量合并到分支。

#### 剪枝算法 Deduce algorithms

- deduce(): 如果实例是在当前变量赋值下满足，它将返回SATISFIABLE; 如果实例包含一个冲突的子句，它将返回CONFLICT; 否则，它会返回UNKNOWN，解算器将继续分支。  **单元子句规则** 似乎是最有效的因为它需要相对较少的计算能力，但可以修剪大的搜索空间。
- 该unit clause规则规定，对于某个clause，如果除了其中一个文字之外的所有条款都已存在如果赋值为0，则必须为其余（未分配的）文字赋值该条款的值为1，这对于满足该公式是必不可少的。这些子句称为 **单元子句** ， **单元子句**中未分配的字符是称为 **单位字面量** (unit liberal)。 调用将值1分配给所有单位文字的过程**单位传播，**或有时称为**布尔约束传播(Boolean Constraint Propagation BCP) **。
- BCP引擎的功能是检测变量赋值后的单元子句和冲突子句。BCP的简单直观实现是为每个子句保留计数器。如果实例有 *m个*子句和*n个*变量，平均每个子句有 *l个*文字，然后每当一个变量被赋值时，就在平均 *lm / n*计数器需要更新。变量赋值的每个撤消也将平均更新 *lm / n*计数器。

#### BCP

给定一个公式和具有DL的分配集，推导出任何必要的赋值及其DL，并通过向初始集添加必要的赋值继续传递这一过程。 必要赋值仅由重复申请*单位子句规则*决定 。没有更多必要的赋值可以推断出时停止，或者在发现冲突时。

为每个子句保留一个计数器，记录该子句包含子句中有多少值0字面量，并在每次子句中的文字设置为0时修改计数器。

*可变状态独立衰减和（VSIDS）*

描述如下：

（1）每个极性中的每个变量都有一个计数器，初始化为0。

（2）当一个子句被添加到数据库时，计数器与子句中的每个文字相关联的递增。

（3）（未分配）变量和极性最高在每个决定中选择计数器。

（4）默认情况下，关系是随机打破的，尽管如此配置

（5）所有计数器定期除以常数。

#### 冲突解决 Conficting sovling

遇到冲突子句时，解算器需要回溯并撤消决定。冲突分析是找出冲突原因的程序试图解决它。它告诉SAT求解器在某个搜索空间中该问题没有解决方案，并指示新的搜索空间以继续搜索。

- 按时间顺序回溯：对于每个决策变量，求解器都会保留一个标志，指示是否已经尝试过两个阶段（即翻转）与否。发生冲突时，进行冲突分析过程查找具有最高决策级别的决策变量翻转，标记它翻转，撤消该决策级别之间的所有任务和当前决策级别，然后尝试决策变量的另一个阶段。

#### 子句存储

线性方式存储(稀疏矩阵)，每个子句占据着自己的空间，不存在重叠子句。早期的指针型数据结构，虽然方便操纵子句数据库(添加/删除操作)，但是会导致大量**缓存未命中**因为缺乏访问地址。阵列数据优点是在内存利用率，占用连续的内存空间，增加了访问位置。

**trie**[SATO]，三元树，Pos，Neg，和DC（正负和不在意），叶节点的值是0或1。从根到叶子代表一个路径/子句。

#### 预处理 重启 和其他

简化实例 加快求解过程。

两个除变量外完全相同的问题可能需要完全不同的时间来通过某个SAT求解器来解决（例如，一个可以在几秒钟内解决，而另一个需要几天）。

**随机重启**来应对这种现象。




## Reformulating

- fair Discrete structures [FDS]
- linear temporal logic[LTL]
- simple programming language[SPL]
- specification
  - algorithmic verification - finite-state:enumerative , symbolic
  - deductive methodology - (need human interaction)



## Alloy

modeling "ceilings and floors"

```python
sig Platform {}
# there are “Platform” things
sig Man {ceiling, floor: Platform}
# each Man has a ceiling and a floor Platform
pred Above(m, n: Man) {m.floor = n.ceiling}
# Man m is “above” Man n if m's floor is n's ceiling
fact {all m: Man | some n: Man | Above (n,m)}
# "One Man's Ceiling Is Another Man's Floor"
```

checking “ceilings and floors”

```python
assert BelowToo {
all m: Man | some n: Man | Above (m,n)
}
# "One Man's Floor Is Another Man's Ceiling"?
check BelowToo for 2
# check "One Man's Floor Is Another Man's Ceiling"
# counterexample with 2 or less platforms and men?
●clicking “Execute” ran this command
–counterexample found, shown in graphic
```

![1547102339464](/tmp/1547102339464.png)