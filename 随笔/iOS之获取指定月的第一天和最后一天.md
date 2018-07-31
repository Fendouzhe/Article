获取指定月的第一天和最后一天
```
+ (NSString *)getMonthBeginAndEndWith:(NSString *)dateStr{    
        
    NSDateFormatter *format=[[NSDateFormatter alloc] init];    
    [format setDateFormat:@"yyyy-MM"];    
    NSDate *newDate=[format dateFromString:dateStr];    
    double interval = 0;    
    NSDate *beginDate = nil;    
    NSDate *endDate = nil;    
    NSCalendar *calendar = [NSCalendar currentCalendar];    
        
    [calendar setFirstWeekday:2];//设定周一为周首日    
    BOOL ok = [calendar rangeOfUnit:NSMonthCalendarUnit startDate:&beginDate interval:&interval forDate:newDate];    
    //分别修改为 NSDayCalendarUnit NSWeekCalendarUnit NSYearCalendarUnit    
    if (ok) {    
        endDate = [beginDate dateByAddingTimeInterval:interval-1];    
    }else {    
        return @"";    
    }    
    NSDateFormatter *myDateFormatter = [[NSDateFormatter alloc] init];    
    [myDateFormatter setDateFormat:@"yyyy.MM.dd"];    
    NSString *beginString = [myDateFormatter stringFromDate:beginDate];    
    NSString *endString = [myDateFormatter stringFromDate:endDate];    
    NSString *s = [NSString stringWithFormat:@"%@-%@",beginString,endString];    
    return s;    
}   
```
方法调用：
```
///调用格式（yyyy-MM）  
 NSString *dateStr = [self getMonthBeginAndEndWith:@"2016-9"];  
```
