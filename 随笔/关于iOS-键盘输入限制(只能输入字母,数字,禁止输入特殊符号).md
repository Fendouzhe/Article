首先我们要设置一下键盘类型
```
 textFiled.keyboardType = UIKeyboardTypeASCIICapable;(根据个人喜好设置键盘)
```
然后我们要设置textfield的代理<UITextFieldDelegate>

设置好代理就开始写键盘了

先来定义几个宏定义
```
#define NUM @"0123456789"
#define ALPHA @"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"
#define ALPHANUM @"ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"
```
然后写代理方法

```
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string
{
    NSCharacterSet *cs = [[NSCharacterSet characterSetWithCharactersInString:ALPHANUM] invertedSet];
    NSString *filtered = [[string componentsSeparatedByCharactersInSet:cs] componentsJoinedByString:@""];
    return [string isEqualToString:filtered];
}
```
注:需要给哪个textfield设置键盘,就给哪个textfield设置代理即可
