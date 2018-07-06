---
title: js数组工具处理之合并相同字段分组
date: 2018-03-05 15:30:18
tags:
       - js
       - 工具
---
#js数组工具处理之合并相同字段分组

话不多说直接上码：

以下是根据time进行分组的
var tempList = [
      { userImage: '', userName: '李测试', userWork: '前端', iPhone: '18516776220', time: '2017-11-17',company:"上海测试一公司"},
      { userImage: '', userName: '刘测试', userWork: '前端', iPhone: '18511111111', time: '2017-11-18', company: "上海测试二公司"},
      { userImage: '', userName: '黄一明', userWork: '前端', iPhone: '18588888888', time: '2017-11-18', company: "上海测试三公司"},
      { userImage: '', userName: '黄大明', userWork: '前端', iPhone: '18588888888', time: '2017-11-18', company: "上海测试五公司" },
      { userImage: '', userName: '董测试', userWork: '法制特', iPhone: '18522222222', time: '2017-11-20', company: "上海测试二公司"}
    ];
 
    var map = {},
        dest = [];
    for (var i = 0; i < tempList.length; i++) {
      var ai = tempList[i];
      if (!map[ai.time]) {
        dest.push({
          initial: ai.time,
          busInfoList: [ai]
        });
        map[ai.time] = ai;
      } else {
        for (var j = 0; j < dest.length; j++) {
          var dj = dest[j];
          if (dj.initial == ai.time) {
            dj.busInfoList.push(ai);
            break;
          }
        }
      }
    }
     console.log(dest);
     
     本文转载： https://blog.csdn.net/jimolangyaleng/article/details/78593873
