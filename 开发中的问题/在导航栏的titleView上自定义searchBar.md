1、初始化一个searchBar，设置好想要的宽、高。
```
UISearchBar *searchCar = [[UISearchBar alloc] initWithFrame:CGRectMake(0, 0, KScreenWidth-40, 34)];
searchCar.delegate = self;
searchCar.placeholder = @"搜索";
searchCar.clipsToBounds = YES;
searchCar.backgroundImage = [UIImage imageWithColor:[UIColor clearColor]];
//searchCar.layer.cornerRadius = 5.f;
//searchCar.layer.borderWidth = 1.0f;
//searchCar.layer.borderColor = LRColor(198, 198, 198).CGColor;
```
2、新建一个searchBar 的background View，并且添加子view ：_searchBar  
```
UIView *searchBgView = [UIView new];
searchBgView.backgroundColor = [UIColor clearColor];
[searchBgView addSubview:searchCar];
```

3、把searchBarBgView 赋值给titleView
```
 self.navigationItem.titleView = searchBgView;
 self.navigationItem.titleView.frame = CGRectMake(0, 0, KScreenWidth-40, 34);
```

**技巧在于_searchBar 作为 searchBarBgView的子View，可以调整自己的大小**。

转载：https://www.jianshu.com/p/207b4b30b58c
