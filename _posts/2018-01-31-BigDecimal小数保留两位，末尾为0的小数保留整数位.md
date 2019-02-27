```
public static void main(String[] args) {
	DecimalFormat df = new DecimalFormat("###.##");
        BigDecimal b1 = new BigDecimal("28.0109");
        BigDecimal b2 = new BigDecimal("28.00");
        System.out.println("小数格式化：" + df.format(b1));
        System.out.println("整数格式化：" + df.format(b2));
}
```