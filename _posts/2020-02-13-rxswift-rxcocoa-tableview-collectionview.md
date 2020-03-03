---
layout: post
title:  "[RxSwift] TableView, CollectionView in RxCocoa"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, rxcocoa]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---

# TableView

기존 UITableView를 통해 데이터를 표시하는 구조를 RxCocoa를 사용하여 구현해보도록 하겠습니다 

우선 TableView에 보여줄 리스트 형태의 데이터를 `Observable`로 변경합니다.

```swift
var nameList = appleProducts.map { $0.name }
var productList = appleProducts
```
```swift
let nameObservable = Observable.of(appleProducts.map { $0.name })
let productObservable = Observable.of(appleProducts)
```

## DataSource

- `UITableViewDataSource`를 따로 구현하지 않고 `items` 메소드로 간단하게 셀을 등록할 수 있습니다.

![/img/rxcocoa-tableview-collectionview/tableview-items.png](/img/rxcocoa-tableview-collectionview/tableview-items.png)

```swift
// #1
nameObservable
    .bind(to: listTableView.rx.items) { (tableView, row, element) in
        let cell = tableView.dequeueReusableCell(withIdentifier: "standardCell")!
        cell.textLabel?.text = element
        return cell
    }
    .disposed(by: bag)

// #2
nameObservable
    .bind(to: listTableView.rx.items(cellIdentifier: "standardCell")) { row, element, cell in
        cell.textLabel?.text = element
    }
    .disposed(by: bag)

// #3
productObservable
    .bind(to: listTableView.rx.items(
        cellIdentifier: "productCell",
        cellType: ProductTableViewCell.self)
    ) { [weak self] row, element, cell in
        cell.categoryLabel.text = element.category
        cell.productNameLabel.text = element.name
        cell.summaryLabel.text = element.summary
        cell.priceLabel.text = self?.priceFormatter.string(for: element.price)
    }
    .disposed(by: bag)
```

## Delegate

`tableView:didSelectRowAtIndexPath:` 대신 `itemSelected`, `modelSelected`를 사용하여 셀 선택 이벤트를 구현합니다.

`var itemSelected: ControlEvent<IndexPath>`

- 셀을 선택 할 때마다 `IndexPath`를 가지고 있는 `next` 이벤트를 방출

```swift
listTableView.rx.itemSelected
    .subscribe(onNext: { [weak self] indexPath in
        self?.listTableView.deselectRow(at: indexPath, animated: true)
    })
    .disposed(by: bag)
```
`func modelSelected<T>(_ modelType: T.Type) -> ControlEvent<T>`

- `modelSelected` 은 `IndexPath` 가 아니라 실제 모델 데이터를 방출

```swift
listTableView.rx.modelSelected(Product.self)
    .subscribe(onNext: { product in
        print(product.name)
    })
    .disposed(by: bag)
```
`zip` 메소드를 활용하여 `modelSelected`, `itemSelected`를 병합하여 모델 데이터와 인덱스를 한꺼번에 방출할 수 있습니다.

```swift
Observable.zip(listTableView.rx.modelSelected(Product.self), listTableView.rx.itemSelected)
    .bind { [weak self] (product, indexPath) in
        self?.listTableView.deselectRow(at: indexPath, animated: true)
        print(product.name)
    }
    .disposed(by: bag)
```

만약 기존처럼 `tableView.delegate = self`로  `Delegate` 를 지정해 준다면 `RxCocoa`의 `Delegate` 메소드는 더이상 동작하지 않습니다.

기존의 `UITableViewDelegate`를 `RxCocoa`와 같이 사용하고 싶다면 아래와 같이 `Delegate`를 지정해 줘야 합니다.

```swift
listTableView.rx.setDelegate(self)
    .disposed(by: bag)
```

# CollectionView

`CollectionView` 의 구현도 `TableView`와 비슷합니다.

리스트 형태의 데이터를 `Observable`로 변경하고 `CollectionView` 에 바인딩 해 줍니다

```swift
let colorObservable = Observable.of(MaterialBlue.allColors)
```

## DataSource

`UICollectionViewDataSource`를 따로 구현하지 않고 `items` 메소드로 간단하게 셀을 등록할 수 있습니다.

![/img/rxcocoa-tableview-collectionview/collectionview-items.png](/img/rxcocoa-tableview-collectionview/collectionview-items.png)

```swift
colorObservable
    .bind(to: listCollectionView.rx
        .items(cellIdentifier: "colorCell", cellType: ColorCollectionViewCell.self)) { index, color, cell in
        cell.backgroundColor = color
        cell.hexLabel.text = color.rgbHexString
    }
    .disposed(by: bag)
```

## Delegate

`collectionView:didSelectItemAtIndexPath:` 대신 `itemSelected`, `modelSelected`를 사용하여 셀 선택 이벤트를 구현합니다.

`var itemSelected: ControlEvent<IndexPath>`

- 셀을 선택 할 때마다 `IndexPath`를 가지고 있는 `next` 이벤트를 방출

```swift
listCollectionView.rx.itemSelected
    .subscribe(onNext: { index in
        print("\(index.section) \(index.row)")
    })
    .disposed(by: bag)
```

`func modelSelected<T>(_ modelType: T.Type) -> ControlEvent<T>`

- `modelSelected` 은 `IndexPath` 가 아니라 실제 모델 데이터를 방출

```swift
listCollectionView.rx.modelSelected(UIColor.self)
    .subscribe(onNext: { color in
        print(color.rgbHexString)
    })
    .disposed(by: bag)
```

`TableView`와 동일하게 기존의 `UICollectionViewDelegate`, `UICollectionViewDelegateFlowLayout` 를 `RxCocoa`와 같이 사용하고 싶다면 아래와 같이 `Delegate`를 지정해 줘야 합니다.

```swift
listCollectionView.rx.setDelegate(self)
    .disposed(by: bag)
```

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/](https://kxcoding.com/)  

