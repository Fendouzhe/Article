####1、 对以下一组数据进行降序排序（冒泡排序）。“24，17，85，13，9，54，76，45，5，63”

```
int main(int argc, char *argv[]) {

    int array[10] = {24, 17, 85, 13, 9, 54, 76, 45, 5, 63};

    int num = sizeof(array)/sizeof(int);

    for(int i = 0; i < num-1; i++) {

        for(int j = 0; j < num - 1 - i; j++) {

            if(array[j] < array[j+1]) {

                int tmp = array[j];

                array[j] = array[j+1];

                array[j+1] = tmp;

            }

        }

    }

    for(int i = 0; i < num; i++) {

        printf("%d", array[i]);

        if(i == num-1) {

            printf("\n");

        }

        else {

            printf(" ");

        }

    }

}
```
####2、 对以下一组数据进行升序排序（选择排序）。“86, 37, 56, 29, 92, 73, 15, 63, 30, 8”

```
void sort(int a[],int n)
{

    int i, j, index;

    for(i = 0; i < n - 1; i++) {

        index = i;

        for(j = i + 1; j < n; j++) {

            if(a[index] > a[j]) {

                index = j;

            }

        }

        if(index != i) {

            int temp = a[i];

            a[i] = a[index];

            a[index] = temp;

        }

    }

}

int main(int argc, const char * argv[]) {

    int numArr[10] = {86, 37, 56, 29, 92, 73, 15, 63, 30, 8};

    sort(numArr, 10);

    for (int i = 0; i < 10; i++) {

        printf("%d, ", numArr[i]);

    }

    printf("\n");

    return 0;

}
```

####3、 快速排序算法

```
void sort(int *a, int left, int right) {

if(left >= right) {

return ;

}

int i = left;

int j = right;

int key = a[left];

while (i < j) {

while (i < j && key >= a[j]) {

j--;

}

a[i] = a[j];

while (i < j && key <= a[i]) {

    i++;

}

a[j] = a[i];

}

a[i] = key;

sort(a, left, i-1);

sort(a, i+1, right);

}
```


####4、 归并排序

```
void merge(int sourceArr[], int tempArr[], int startIndex, int midIndex, int endIndex) {

    int i = startIndex;

    int j = midIndex + 1;

    int k = startIndex;

    while (i != midIndex + 1 && j != endIndex + 1) {

        if (sourceArr[i] >= sourceArr[j]) {

            tempArr[k++] = sourceArr[j++];

        } else {

            tempArr[k++] = sourceArr[i++];

        }

    }

    while (i != midIndex + 1) {

        tempArr[k++] = sourceArr[i++];

    }

    while (j != endIndex + 1) {

        tempArr[k++] = sourceArr[j++];

    }

    for (i = startIndex; i <= endIndex; i++) {

        sourceArr[i] = tempArr[i];

    }

}


void sort(int souceArr[], int tempArr[], int startIndex, int endIndex) {

    int midIndex;

    if (startIndex < endIndex) {

        midIndex = (startIndex + endIndex) / 2;

        sort(souceArr, tempArr, startIndex, midIndex);

        sort(souceArr, tempArr, midIndex + 1, endIndex);

        merge(souceArr, tempArr, startIndex, midIndex, endIndex);

    }

}


int main(int argc, const char * argv[]) {

    int numArr[10] = {86, 37, 56, 29, 92, 73, 15, 63, 30, 8};

    int tempArr[10];

    sort(numArr, tempArr, 0, 9);

    for (int i = 0; i < 10; i++) {

        printf("%d, ", numArr[i]);

    }

    printf("\n");

    return 0;

}
```

####5、 实现二分查找算法（编程语言不限）

```
int bsearchWithoutRecursion(int array[],int low,int high,int target) {

while(low <= high) {

int mid = (low + high) / 2;

if(array[mid] > target)

high = mid - 1;

else if(array[mid] < target)

low = mid + 1;

else    //findthetarget

return mid;

}

//the array does not contain the target

return -1;

}

----------------------------------------

递归实现

int binary_search(const int arr[],int low,int high,int key)
{

int mid=low + (high - low) / 2;

if(low > high)

return -1;

else{

if(arr[mid] == key)

return mid;

else if(arr[mid] > key)

return binary_search(arr, low, mid-1, key);

else

return binary_search(arr, mid+1, high, key);

}

}
```

####6、 如何实现链表翻转（链表逆序）？ 
思路：每次把第二个元素提到最前面来。

```
#include <stdio.h>

#include <stdlib.h>


typedef struct NODE {

    struct NODE *next;

    int num;

}node;


node *createLinkList(int length) {

    if (length <= 0) {

        return NULL;

    }

    node *head,*p,*q;

    int number = 1;

    head = (node *)malloc(sizeof(node));

    head->num = 1;

    head->next = head;

    p = q = head;

    while (++number <= length) {

        p = (node *)malloc(sizeof(node));

        p->num = number;

        p->next = NULL;

        q->next = p;

        q = p;

    }

    return head;
}


void printLinkList(node *head) {

    if (head == NULL) {

        return;

    }

    node *p = head;

    while (p) {

        printf("%d ", p->num);

        p = p -> next;

    }

    printf("\n");

}


node *reverseFunc1(node *head) {

    if (head == NULL) {

        return head;


    }


    node *p,*q;

    p = head;

    q = NULL;

    while (p) {

        node *pNext = p -> next;

        p -> next = q;

        q = p;

        p = pNext;

    }

    return q;

}


int main(int argc, const char * argv[]) {

    node *head = createLinkList(7);

    if (head) {

        printLinkList(head);

        node *reHead = reverseFunc1(head);

        printLinkList(reHead);

        free(reHead);

    }

    free(head);

    return 0;

}
```

####7、 实现一个字符串“how are you”的逆序输出（编程语言不限）。如给定字符串为“hello world”,输出结果应当为“world hello”。

```
int spliterFunc(char *p) {

    char c[100][100];

    int i = 0;

    int j = 0;


    while (*p != '\0') {

        if (*p == ' ') {

            i++;

            j = 0;

        } else {

            c[i][j] = *p;

            j++;

        }

        p++;


    }


    for (int k = i; k >= 0; k--) {

        printf("%s", c[k]);

        if (k > 0) {

            printf(" ");

        } else {

            printf("\n");

        }

    }

    return 0;


}
```

####8、 给定一个字符串，输出本字符串中只出现一次并且最靠前的那个字符的位置？如“abaccddeeef”,字符是b,输出应该是2。

```
char *strOutPut(char *);


int compareDifferentChar(char, char *);


int main(int argc, const char * argv[]) {


    char *inputStr = "abaccddeeef";

    char *outputStr = strOutPut(inputStr);

    printf("%c \n", *outputStr);

    return 0;

}


char *strOutPut(char *s) {

    char str[100];

    char *p = s;

    int index = 0;

    while (*s != '\0') {

        if (compareDifferentChar(*s, p) == 1) {

            str[index] = *s;

            index++;

        }

        s++;

    }

    return &str;
}


int compareDifferentChar(char c, char *s) {

    int i = 0;

    while (*s != '\0' && i<= 1) {

        if (*s == c) {

            i++;

        }

        s++;
    }

    if (i == 1) {

        return 1;

    } else {

        return 0;

    }

}
```

####9、 二叉树的先序遍历为FBACDEGH,中序遍历为：ABDCEFGH,请写出这个二叉树的后序遍历结果。

ADECBHGF

先序+中序遍历还原二叉树：先序遍历是：ABDEGCFH 中序遍历是：DBGEACHF
首先从先序得到第一个为A，就是二叉树的根，回到中序，可以将其分为三部分：

左子树的中序序列DBGE，根A，右子树的中序序列CHF

接着将左子树的序列回到先序可以得到B为根，这样回到左子树的中序再次将左子树分割为三部分：

左子树的左子树D，左子树的根B，左子树的右子树GE

同样地，可以得到右子树的根为C

类似地将右子树分割为根C，右子树的右子树HF，注意其左子树为空

如果只有一个就是叶子不用再进行了，刚才的GE和HF再次这样运作，就可以将二叉树还原了。

####10、 打印2-100之间的素数。

```
int main(int argc, const char * argv[]) {

    for (int i = 2; i < 100; i++) {

        int r = isPrime(i);

        if (r == 1) {

            printf("%ld ", i);

        }

    }

    return 0;

}


int isPrime(int n)
{

    int i, s;

    for(i = 2; i <= sqrt(n); i++)

        if(n % i == 0)  return 0;

    return 1;

}
```

####11、 求两个整数的最大公约数。

```
int gcd(int a, int b) {

    int temp = 0;

    if (a < b) {

        temp = a;

        a = b;

        b = temp;

    }

    while (b != 0) {

        temp = a % b;

        a = b;

        b = temp;

    }

    return a;

}
```
