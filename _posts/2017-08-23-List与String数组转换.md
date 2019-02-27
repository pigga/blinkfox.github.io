List 转换为 String数组
```
List<String> list = new ArrayList<String>();    
list.add("a1");    
list.add("a2");    
String[] toBeStored = list.toArray(new String[list.size()]);  
```
String数组转List
```
String[] arr = new String[] {"1", "2"};  
List list = Arrays.asList(arr);
```