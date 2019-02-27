jQuery获取Select选择的Text和Value:<br/>
语法解释：
```
1. $("#select_id").change(function(){//code...});   //为Select添加事件，当选择其中一项时触发
2. var checkText=$("#select_id").find("option:selected").text();  //获取Select选择的Text
3. var checkValue=$("#select_id").val();  //获取Select选择的Value
4. var checkIndex=$("#select_id ").get(0).selectedIndex;  //获取Select选择的索引值
5. var maxIndex=$("#select_id option:last").attr("index");  //获取Select最大的索引值
 ```
jQuery设置Select选择的Text和Value:<br/>
语法解释：
```
1. $("#select_id ").get(0).selectedIndex=1;  //设置Select索引值为1的项选中
2. $("#select_id ").val(4);   //设置Select的Value值为4的项选中
3. $("#select_id option[text='jQuery']").attr("selected", true);   //设置Select的Text值为jQuery的项选中
```
jQuery添加/删除Select的Option项：<br/>
点击一次，Select将追加一个Option<br/>
点击将在Select第一个位置插入一个Option<br/>
点击将删除Select中索引值最大Option(最后一个)<br/>
语法解释：
```
1. $("#select_id").append("<option value='Value'>Text</option>");  //为Select追加一个Option(下拉项)
2. $("#select_id").prepend("<option value='0'>请选择</option>");  //为Select插入一个Option(第一个位置)
3. $("#select_id option:last").remove();  //删除Select中索引值最大Option(最后一个)
4. $("#select_id option[index='0']").remove();  //删除Select中索引值为0的Option(第一个)
5. $("#select_id option[value='3']").remove();  //删除Select中Value='3'的Option
5. $("#select_id option[text='4']").remove();  //删除Select中Text='4'的Option
```
```
function jsSelectIsExitItem(objSelect, objItemValue) { 
    var isExit = false;
    for (var i = 0; i < objSelect.options.length; i++) {
        if (objSelect.options[i].value == objItemValue) {
            isExit = true;
            break;
        }
    }
    return isExit;
}            
// 2.向select选项中 加入一个Item        
function jsAddItemToSelect(objSelect, objItemText, objItemValue) {
    //判断是否存在
    if (jsSelectIsExitItem(objSelect, objItemValue)) { 
        alert("该Item的Value值已经存在");
    } else {
        var varItem = new Option(objItemText, objItemValue);
        objSelect.options.add(varItem);
        alert("成功加入");
    }
}
// 3.从select选项中 删除一个Item        
function jsRemoveItemFromSelect(objSelect, objItemValue) {
    //判断是否存在
    if (jsSelectIsExitItem(objSelect, objItemValue)) {
        for (var i = 0; i < objSelect.options.length; i++) {
            if (objSelect.options[i].value == objItemValue) {
                objSelect.options.remove(i);
                break;
            }
        }
        alert("成功删除");
    } else {
        alert("该select中 不存在该项");
    }
}
// 4.删除select中选中的项    
function jsRemoveSelectedItemFromSelect(objSelect) { 
    var length = objSelect.options.length - 1;        
    for(var i = length; i >= 0; i--){            
        if(objSelect[i].selected == true){                
            objSelect.options[i] = null;            
        }        
    }    
}         
// 5.修改select选项中 value="paraValue"的text为"paraText" 
function jsUpdateItemToSelect(objSelect, objItemText, objItemValue) {
    //判断是否存在            
    if (jsSelectIsExitItem(objSelect, objItemValue)) {
        for (var i = 0; i < objSelect.options.length; i++) {
            if (objSelect.options[i].value == objItemValue) {
                objSelect.options[i].text = objItemText; 
                break;                    
            }                
        }                
        alert("成功修改");            
    } else {
        alert("该select中 不存在该项");
    }
}           
// 6.设置select中text="paraText"的第一个Item为选中        
function jsSelectItemByValue(objSelect, objItemText) {
    //判断是否存在
    var isExit = false;
    for (var i = 0; i < objSelect.options.length; i++) {
        if (objSelect.options[i].text == objItemText) {
            objSelect.options[i].selected = true; 
            isExit = true;
            break;                
        }            
    }                  
    //Show出结果            
    if (isExit) {                
        alert("成功选中");            
    } else {                
        alert("该select中 不存在该项");            
    }        
}           
// 7.设置select中value="paraValue"的Item为选中    
document.all.objSelect.value = objItemValue;           
// 8.得到select的当前选中项的value    
var currSelectValue = document.all.objSelect.value;
// 9.得到select的当前选中项的text
var currSelectText = document.all.objSelect.options[document.all.objSelect.selectedIndex].text;           
// 10.得到select的当前选中项的Index    
var currSelectIndex = document.all.objSelect.selectedIndex;
// 11.清空select的项    
document.all.objSelect.options.length = 0;
```