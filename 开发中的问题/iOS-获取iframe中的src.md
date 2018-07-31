公司项目在线学习模块需要用webView播放在线培训视频，后台返回视频的iframe并没有直接返回视频的url给我们，这需要我们自己去截取iframe里的src传给webView播放，后台返回的格式总共有3种类型如下：
```
videoPath = <iframe height=498 width=510 src=http://player.youku.com/embed/XMzE1OTMzOTM3Mg== frameborder=0 'allowfullscreen'></iframe>;

videoPath = <iframe height=498 width=510 src='http://player.youku.com/embed/XMzEzNzc3Mjc4MA==' frameborder=0 'allowfullscreen'></iframe>;

videoPath = <iframe src="http://open.iqiyi.com/developer/player_js/coopPlayerIndex.html?vid=69d0de5269cd1f9061049f231f23745f&tvId=829999100&accessToken=2.f22860a2479ad60d8da7697274de9346&appKey=3955c3425820435e86d0f4cdfe56f5e7&appId=1368&height=100%&width=100%" frameborder="0" allowfullscreen="true" width="100%" height="100%"></iframe>;
```
我这里用谓词获取html中iframe里的src，抽成一个方法直接调用即可：
```
-(NSString*)getIframeSrcWithHtml:(NSString *)htmlText{
    
    if (htmlText == nil) {
        return nil;
    }

    NSError *error;
    NSString *regulaStr = @"<iframe[^>]+src\\s*=\\s*['\"]([^'\"]+)['\"][^>]*>";
    NSRegularExpression *regex = [NSRegularExpression regularExpressionWithPattern:regulaStr
                                                                           options:NSRegularExpressionCaseInsensitive
                                                                             error:&error];
    NSArray *arrayOfAllMatches = [regex matchesInString:htmlText options:0 range:NSMakeRange(0, [htmlText length])];
    
    NSMutableArray *resultArray = [NSMutableArray array];
    
    for (NSTextCheckingResult *item in arrayOfAllMatches) {
        
        NSString *imgHtml = [htmlText substringWithRange:[item rangeAtIndex:0]];
        
        NSArray *tmpArray = nil;
        if ([imgHtml rangeOfString:@"src="].location != NSNotFound) {
            tmpArray = [imgHtml componentsSeparatedByString:@"src="];
        } else if ([imgHtml rangeOfString:@"src='"].location != NSNotFound) {
            tmpArray = [imgHtml componentsSeparatedByString:@"src='"];
        } else if ([imgHtml rangeOfString:@"src=\""].location != NSNotFound) {
            tmpArray = [imgHtml componentsSeparatedByString:@"src=\""];
        } 
        
        if (tmpArray.count >= 2) {
            NSString *src = tmpArray[1];
            NSArray *strArr = [src componentsSeparatedByString:@" "];
            NSString *srcString = strArr.firstObject;
            srcString = [srcString stringByReplacingOccurrencesOfString:@"\"" withString:@""];
            srcString = [srcString stringByReplacingOccurrencesOfString:@"'" withString:@""];
            srcString = [srcString stringByTrimmingCharactersInSet:[NSCharacterSet whitespaceAndNewlineCharacterSet]];
            [resultArray addObject:srcString];
        }
    }
    return resultArray.firstObject;
}
```
