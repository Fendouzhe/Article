```
    NSString *testString = @"中文688llrLLR";
    NSInteger alength = [testString length];

    for (int i = 0; i<alength; i++) {
        char commitChar = [testString characterAtIndex:i];
        NSString *temp = [testString substringWithRange:NSMakeRange(i,1)];
        const char *u8Temp = [temp UTF8String];
        if (3==strlen(u8Temp)){

            NSLog(@"字符串中含有中文");
        }else if((commitChar>64)&&(commitChar<91)){

            NSLog(@"字符串中含有大写英文字母");
        }else if((commitChar>96)&&(commitChar<123)){

            NSLog(@"字符串中含有小写英文字母");
        }else if((commitChar>47)&&(commitChar<58)){

            NSLog(@"字符串中含有数字");
        }else{

            NSLog(@"字符串中含有非法字符");
        }
    }
```

```
//判断是否为整形：

- (BOOL)isPureInt:(NSString*)string{
    NSScanner* scan = [NSScanner scannerWithString:string];
    int val;
    return[scan scanInt:&val] && [scan isAtEnd];
}
//判断是否为浮点形：
- (BOOL)isPureFloat:(NSString*)string{
    NSScanner* scan = [NSScanner scannerWithString:string];
    float val;
    return[scan scanFloat:&val] && [scan isAtEnd];
}
```
