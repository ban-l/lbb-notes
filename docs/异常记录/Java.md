## MapStruct注意事项

使用`MapStruct`用于POJO之间的相互转换时，若源类含有`getXXX`形式的方法，没有`XXX`字段，则会报错。

所以使用时，注意方法命名。