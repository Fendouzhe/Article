原文作者：艾欧艾斯之手
原文链接：https://www.jianshu.com/p/2ffca6f93c1c

我们都知道UICollectionViewFlowLayout有一个minimumInteritemSpacing属性可以控制cell之间水平的间距，但是这个属性并不是你设置成多少它的间距一定是多少，从这个单词的字面意思就可以看出来它指的是cell之间的最小间距。也就是说cell的间距是大大于或者等于这个属性的。于是乎，最近就碰到了测试丢过来的问题-UICollectionViewCell之间的布局。

#### 问题重现

关键代码:

```
let layout = UICollectionViewFlowLayout()
layout.minimumLineSpacing = 8.0
layout.minimumInteritemSpacing = 8.0
layout.sectionInset = UIEdgeInsets(top: 25, left: 15, bottom: 30, right: 15)
let collection = UICollectionView(frame: self.view.bounds, collectionViewLayout: layout)
collection.backgroundColor = UIColor.white
collection.delegate = self
collection.dataSource = self
collection.register(UINib(nibName: "CollectionCell", bundle: Bundle.main),forCellWithReuseIdentifier: "CollectionCell")
self.view.addSubview(collection)

```

以上关键代码是layout.minimumInteritemSpacing = 8.0,把cell间距设置成8之后，本以为就可以高枕无忧了，但是载入数据之后数据效果如下图:

![](https://upload-images.jianshu.io/upload_images/9610202-a78c0fb148f6543c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


如图中红框内的间距明显不是我们想要。

#### 明确需求

现在想要达到的效果是，cell全部向左对齐，间距一定要控制在8.0，不够就换一行显示。

#### 如何解决

要达到这样的效果就只能是修改布局了。先来看一下布局的实现:

```
//
//  DVMaximumSpacingLayout.swift
//  Amall
//  一个可以设置cell之间最大间距的布局,用于商品详情的属性
//  Created by David Yu on 2018/4/26.
//  Copyright © 2018年 David. All rights reserved.
//

import UIKit

// MARK:-   DVMaximumSpacingLayout代理
@objc protocol DVMaximumSpacingLayoutDelegate {

    func collectionView(_ collectionView: UICollectionView?, layout collectionViewLayout: DVMaximumSpacingLayout, sizeForItemAt indexPath: IndexPath) -> CGSize
    @objc optional func collectionView(_ collectionView: UICollectionView?, layout collectionViewLayout: DVMaximumSpacingLayout, referenceSizeForHeaderInSection section: Int) -> CGSize
    @objc optional func collectionView(_ collectionView: UICollectionView?, layout collectionViewLayout: DVMaximumSpacingLayout, referenceSizeForFooterInSection section: Int) -> CGSize

}

class DVMaximumSpacingLayout: UICollectionViewLayout {

    /// cell最大水平间距
    var MaximumSpacing: CGFloat = 0.0
    /// cell竖直间距
    var minimumLineSpacingForSection: CGFloat = 0.0
    /// 间距
    var sectionEdgeInsets: UIEdgeInsets = UIEdgeInsets(top: 0, left: 0, bottom: 0, right: 0)
    /// cell布局
    var cellAttributes = [IndexPath: UICollectionViewLayoutAttributes]()
    /// 头视图布局
    var headerAttributes = [IndexPath: UICollectionViewLayoutAttributes]()
    /// 尾视图布局
    var footerAttributes = [IndexPath: UICollectionViewLayoutAttributes]()
    /// 代理
    var delegate: DVMaximumSpacingLayoutDelegate?
    /// 当前Y坐标
    var currentY : CGFloat = 0

    // MARK:- prepareLayout是一个必须要实现的方法，该方法的功能是为布局提供一些必要的初始化参数
    override func prepare() {
        super.prepare()
        cellAttributes.removeAll()
        headerAttributes.removeAll()
        footerAttributes.removeAll()
        currentY = 0

        //  一共有多少section
        let sectionNum = self.collectionView?.numberOfSections ?? 0
        for i in 0..<sectionNum {
            let supplementaryViewIndex = IndexPath(row: 0, section: i)
            //  计算设置每个header的布局对象
            let headerAttribute = UICollectionViewLayoutAttributes(forSupplementaryViewOfKind: UICollectionElementKindSectionHeader, with: supplementaryViewIndex)
            let headerSize = delegate?.collectionView?(collectionView, layout: self, referenceSizeForHeaderInSection: i) ?? CGSize(width: 0, height: 0)
            headerAttribute.frame = CGRect(origin: CGPoint(x: 0, y: currentY), size: headerSize)
            headerAttributes[supplementaryViewIndex] = headerAttribute
            currentY = headerAttribute.frame.maxY + sectionEdgeInsets.top

            //  计算设置每个cell的布局对象
            //  该section一共有多少row
            let rowNum = self.collectionView?.numberOfItems(inSection: i) ?? 0
            var currentX = sectionEdgeInsets.left
            for j in 0..<rowNum {
                let cellIndex = IndexPath(row: j, section: i)
                let cellAttribute = UICollectionViewLayoutAttributes(forCellWith: cellIndex)
                let cellSize = delegate?.collectionView(collectionView, layout: self, sizeForItemAt: cellIndex) ?? CGSize(width: 0, height: 0)
                if currentX + cellSize.width + sectionEdgeInsets.right > collectionView?.frame.width ?? 0 {
                    //  超过collectview换行,并且collectionview的高度增加
                    currentX = sectionEdgeInsets.left
                    currentY = currentY + cellSize.height + minimumLineSpacingForSection
                }
                cellAttribute.frame = CGRect(origin: CGPoint(x: currentX, y: currentY), size: cellSize)
                currentX = currentX + cellSize.width + MaximumSpacing
                cellAttributes[cellIndex] = cellAttribute

                if j == rowNum - 1 {
                    currentY = currentY + cellSize.height + sectionEdgeInsets.bottom
                }
            }

            //  计算每个footer的布局对象
            let footerAttribute = UICollectionViewLayoutAttributes(forSupplementaryViewOfKind: UICollectionElementKindSectionFooter, with: supplementaryViewIndex)
            let footerSize = delegate?.collectionView?(collectionView, layout: self, referenceSizeForFooterInSection: i) ?? CGSize(width: 0, height: 0)
            footerAttribute.frame = CGRect(origin: CGPoint(x: 0, y: currentY), size: footerSize)
            footerAttributes[supplementaryViewIndex] = footerAttribute
            currentY = currentY + footerSize.height
        }

    }

    // MARK:- 当前屏幕可见的cell、header、footer的布局
    override func layoutAttributesForElements(in rect: CGRect) -> [UICollectionViewLayoutAttributes]? {
        var attributes = [UICollectionViewLayoutAttributes]()
        //  添加当前屏幕可见的cell的布局
        for element in cellAttributes.values {
            if rect.contains(element.frame) {
                attributes.append(element)
            }
        }
        //  添加当前屏幕可见的头视图的布局
        for element in headerAttributes.values {
            if rect.contains(element.frame) {
                attributes.append(element)
            }
        }
        //  添加当前屏幕可见的尾部的布局
        for element in footerAttributes.values {
            if rect.contains(element.frame) {
                attributes.append(element)
            }
        }
        return attributes
    }

    // MARK:- 该方法是为每个Cell返回一个对应的Attributes，我们需要在该Attributes中设置对应的属性，如Frame等
    override func layoutAttributesForItem(at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        return cellAttributes[indexPath]
    }

    // MARK:- 该方法是为每个头和尾返回一个对应的Attributes，我们需要在该Attributes中设置对应的属性，如Frame等
    override func layoutAttributesForSupplementaryView(ofKind elementKind: String, at indexPath: IndexPath) -> UICollectionViewLayoutAttributes? {
        var attr: UICollectionViewLayoutAttributes?
        if elementKind == UICollectionElementKindSectionHeader {
            attr = headerAttributes[indexPath]
        } else {
            attr = footerAttributes[indexPath]
        }
        return attr
    }

    // MARK:- 设置滚动范围
    override var collectionViewContentSize: CGSize {
        let width = collectionView?.frame.width ?? 0
        return CGSize(width: width, height: currentY)
    }

}

```

然后使用几乎和原先差不多，只是布局的代理方法需要更换成自定义布局的代理方法

```
//
//  ViewController.swift
//  UICollectionLayout
//
//  Created by David Yu on 2018/4/27.
//  Copyright © 2018年 David Yu. All rights reserved.
//

import UIKit

class ViewController: UIViewController {

    let titles = ["测试测试测试测试","测试测试测试测试","试测试试测试试测试试测试","试测试试测试试测试试测试试测试试测试","试测试试测试试测试试测试","试测试试测试试测试试测试试测试","试测试试测试试测试试测试试测试试测试","试测试试测试试测试试测试","试测试试测试试测试","试测试试测试试测试试测试试测试","试测试试测试试测试试测试","试测试试测试试测试","试测试","试测试试测试试测试试测试试测试试测试试测试","试测试试测试试测试试测试试测试","试测试试测试试测试"]
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.
        setView()
    }

    func setView() {
        self.view.backgroundColor = UIColor.white
        let layout = DVMaximumSpacingLayout()
        layout.MaximumSpacing = 12.0
        layout.minimumLineSpacingForSection = 8.0
        layout.sectionEdgeInsets = UIEdgeInsets(top: 12, left: 15, bottom: 12, right: 15)
        layout.delegate = self
        let collection = UICollectionView(frame: CGRect(x: 0, y: 64, width: self.view.frame.width, height: self.view.frame.height-64), collectionViewLayout: layout)
        collection.backgroundColor = UIColor.white
        collection.delegate = self
        collection.dataSource = self
        collection.register(UINib(nibName: "CollectionCell", bundle: Bundle.main), forCellWithReuseIdentifier: "CollectionCell")
        self.view.addSubview(collection)
    }

}

extension ViewController: UICollectionViewDelegate, UICollectionViewDataSource, DVMaximumSpacingLayoutDelegate {

    func collectionView(_ collectionView: UICollectionView, numberOfItemsInSection section: Int) -> Int {
        return titles.count
    }

    func collectionView(_ collectionView: UICollectionView, cellForItemAt indexPath: IndexPath) -> UICollectionViewCell {
        let cell = collectionView.dequeueReusableCell(withReuseIdentifier: "CollectionCell", for: indexPath) as! CollectionCell
        cell.title = titles[indexPath.row]
        return cell
    }

    func collectionView(_ collectionView: UICollectionView?, layout collectionViewLayout: DVMaximumSpacingLayout, sizeForItemAt indexPath: IndexPath) -> CGSize {
        let title = titles[indexPath.row]
        let size = title.getSize(UIFont.systemFont(ofSize: 17), size: CGSize(width: UIScreen.main.bounds.width, height: 26))
        return CGSize(width: size.width + 20, height: 26)
    }

}

```

来看看运行效果图:

![](https://upload-images.jianshu.io/upload_images/9610202-2deece2701503c4b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


确实是达到了想要的效果，希望可以帮助到有同样需求的同学。
