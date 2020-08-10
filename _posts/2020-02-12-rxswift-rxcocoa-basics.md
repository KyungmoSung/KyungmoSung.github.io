---
layout: post
title:  "[RxSwift] RxCocoa Basics"
author: Sung Kyungmo
catalog: true
tags: [ios, swift, rxswift, rxcocoa]
comments: true
header-img:   "img/title-bg/rxswift.jpg"
header-mask:  0.6
---

# RxCocoa

- Cocoa Framework에 Reactive의 장점을 더해주는 라이브러리

# Install

```swift
# Podfile
use_frameworks!

target 'YOUR_TARGET_NAME' do
    pod 'RxCocoa'
end
```

## Cocoa Touch

```swift
@IBOutlet weak var valueLabel: UILabel!
   
@IBAction func onTap(_ sender: Any) {
    valueLabel.text = "Hello, Cocoa Touch"
}
```

## RxCocoa

```swift
let disposeBag = DisposeBag()
    
@IBOutlet weak var valueLabel: UILabel!
    
@IBOutlet weak var tapButton: UIButton!
    
override func viewDidLoad() {
    super.viewDidLoad()
      
    tapButton.rx.tap
        .map{ "Hello, RxCocoa" }
        .bind(to: valueLabel.rx.text)
        .disposed(by: disposeBag)
}
```

# Binding

- `Observable` 타입을 채용한 모든 형식이 생산자(Producer)
- `Label` 이나 `ImageView` 같은 UI 컴포넌트는 소비자(Consumer)
- 생산자가 생산한 데이터는 소비자한테 전달되고 소비자는 적절한 방법으로 데이터를 사용함
- Binder는 UI Binding에 사용되는 특별한 `Observer`로 데이터 소비자의 역할을 수행
- Binder는 `Error` 이벤트를 받지 않고 `Next`, `Completed` 이벤트만 전달
- `Main Thread` 에서 실행되는것을 보장<br><br>

  **Cocoa Touch**
- Delegate 패턴을 구현해야함
- 최종 문자열을 조합하는 코드 필요
```swift
  class BindingCocoaTouchViewController: UIViewController {
     @IBOutlet weak var valueLabel: UILabel!
     @IBOutlet weak var valueField: UITextField!
     
     override func viewDidLoad() {
        super.viewDidLoad()
        
        valueLabel.text = ""
        valueField.delegate = self
        valueField.becomeFirstResponder()
     }
     
     override func viewWillDisappear(_ animated: Bool) {
        super.viewWillDisappear(animated)
        
        valueField.resignFirstResponder()
     }
  }
  
  extension BindingCocoaTouchViewController: UITextFieldDelegate {
     func textField(_ textField: UITextField, shouldChangeCharactersIn range: NSRange, replacementString string: String) -> Bool {
        guard let currentText = textField.text else {
           return true
        }
        
        let finalText = (currentText as NSString).replacingCharacters(in: range, with: string)
        valueLabel.text = finalText
        
        return true
     }
  }
```
<br>

  **RxCocoa**
- 코드가 간결해짐
- 최종 문자열을 조합하는 코드 필요 없음
- Delegate 패턴을 구현할 필요가 없어 데이터 흐름을 쉽게 파악 가능
```swift
  class BindingRxCocoaViewController: UIViewController {
    @IBOutlet weak var valueLabel: UILabel!
    @IBOutlet weak var valueField: UITextField!
  
    let disposeBag = DisposeBag()
  
    override func viewDidLoad() {
      super.viewDidLoad()
  
      valueLabel.text = ""
      valueField.becomeFirstResponder()
  
  //    valueField.rx.text
  //      .observeOn(MainScheduler.instance) //메인 스레드에서 실행해야함
  //      .subscribe(onNext: { [weak self] str in
  //        self?.valueLabel.text = str
  //      })
  //      .disposed(by: disposeBag)
  
      //Main Thread 실행을 보장하는 bind를 통해 간결하게 구현 가능
      valueField.rx.text
        .bind(to: valueLabel.rx.text)
        .disposed(by: disposeBag)
    }
  
    override func viewWillDisappear(_ animated: Bool) {
      super.viewWillDisappear(animated)
  
      valueField.resignFirstResponder()
    }
  }
```

## Reference
--- 
> [http://reactivex.io/](http://reactivex.io/)  
> [https://kxcoding.com/](https://kxcoding.com/)  

