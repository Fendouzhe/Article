打开文件的方法：
1.获取文件的沙盒路径path
2.将path路径转化URL
3.用webView显示出来
```
#import "WebViewController.h"

@interface WebViewController ()<UIWebViewDelegate>
@property(nonatomic, strong)UIWebView *webView;
@end

@implementation WebViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    _webView = [[UIWebView alloc] initWithFrame:CGRectMake(0, 0, self.view.frame.size.width, self.view.frame.size.height - 64)];
    _webView.delegate = self;
    _webView.scalesPageToFit = YES;
    [self.view addSubview:_webView];
    
    //获取文件路径
    NSFileManager *manager = [NSFileManager defaultManager];
    NSArray *array = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    //获取document文件夹路径
    NSString *documents = [array lastObject];
    //拼接绝对路径
    NSString *documentPath = [documents stringByAppendingPathComponent:@"myReport/file"];
    //存数据的具体文件夹
    // [manager createDirectoryAtPath:documentPath withIntermediateDirectories:YES attributes:nil error:nil];
    //得到文件名
    NSArray *fileNameArray = [manager subpathsAtPath:documentPath];

    NSString *path = [NSString stringWithFormat:@"%@/%@", documentPath, fileNameArray[0]];

    // NSString *path = [[NSBundle mainBundle] pathForResource:documentName ofType:nil];
    NSURL *url = [NSURL fileURLWithPath:path];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [_webView loadRequest:request];
    
}
```
