---
tId: 1605031
layout: post
title: "数据库相关测试驱动开发实践"
date: 2016-05-03 18:00:00 +0800
categories: TDD 测试
codelang: csharp
desc: "测试驱动开发能够很好的提高系统开发效率，而有关数据库方面的用例，由于数据逻辑关系往往处于变化之中,反而不太容易编写，按照自己的工作实践中的整理，总结了有关数据库方面的测试驱动开发实践的三种用例编写方法，以便于思考和回顾"
---
尽管很早就了解到了测试驱动开发的思路，但是真正的开始实践，还是在最近新启动的一个项目中。

### 背景介绍 ###
测试驱动的项目，按照官方的解释，是首先编写功能测试代码，并根据测试业务逻辑，进行反向编码实现。
我自身在这方面实践较少，测试驱动开发更是首次运行到实践中，对于这种方式并没有更好的感知。反而，因为之前都是直接编码进行业务开发，反而更习惯通过首先编码，其次实现测试逻辑的方式进行。我想这应该也算是一个比较好的过渡方案，如果这种方式能够保证编码质量，满足我们的需要，也是可行的，同样这种方式，也便于我从实际业务的角度去思考系统架构和设计的思路。
当然，如果大家有好的测试驱动实践，请不吝赐教。

### 开发实践 ###
1 强制失败，驱动用例适应性修改
```
[TestCase("1,2,3,4,5,6,7,8,9,112")]
public void FindIdsExist(string ids)
{
    BLL.News.NewsCategoryBLL bll = new News.NewsCategoryBLL();
    var listResult = bll.FindIdsExist(ids);
    Assert.IsNotNull(listResult);
    if (listResult.Count > 0)
    {
          Assert.IsTrue(ids.Contains(listResult[0].ToString()));
    }
    else {
          Assert.IsEmpty("集合数量为0，请修改用例");
    }
}
```
如果测试用例id不存在，通过Assert.IsEmpty输出的错误信息进行提示（亦可以采用其他方法），强制测试失败，来适当调整用例。

2 循环查询法

设定一个循环次数上限，通过查询获取特定的数据，用来进行测试判断
```
public void UpdateStatus()
{
    IProjectApplyBLL iProjectApply = EngineContext.CurrentContainerManager.Resolve<IProjectApplyBLL>();
    Model.ProjectApply pa = null;
    for (int i = 0; i < 1000; i++)
    {
        pa = iProjectApply.Find(i);
        if (pa != null) break;
    }
    Assert.IsNotNull(pa);
    int result = iProjectApply.UpdateStatus(pa.ApplyId, 2);
    Assert.IsTrue(result > 0);
    pa = iProjectApply.Find(pa.ApplyId);
    Assert.IsNotNull(pa);
    Assert.AreEqual((int)pa.Status, 2);
}
```
此方法仅限于主键id可循环预测的情况（int自增长），上述代码通过设定1000的循环上限，查询id在0~1000范围内的第一个数据对象（可结合其他条件），通过Assert断言逻辑进行判断，以便于能够正确执行所需要的测试操作。

3 逻辑断言法

此方法反而是基础，通过断言进行逻辑判断，使测试用例满足特定的逻辑分支以及各种前置或后置条件，具体用法参考上例代码实现。

### 优势 ###
* 通过测试用例，保证功能业务逻辑符合需求预期，对需求进行初步验证
* 减少软件后期测试人员介入的压力，减少项目bug，提升质量
* 测试代码具有代码的自解释性
* 方便维护重构，及时感知受影响业务代码

### 参考资料 ###
浅谈测试驱动开发（TDD）
http://www.ibm.com/developerworks/cn/linux/l-tdd/
