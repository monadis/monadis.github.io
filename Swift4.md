# Swift 4

- private프로퍼티를 extension내에서 접근 가능해짐.
- 클래스와 프로토콜을 조합한 타입이 가능해짐.

```swift
protocol Shakeable { }
func shake() 
extension UIButton: Shakeable { /* ... */ } 
extension UISlider: Shakeable { /* ... */ }
func shakeEm(controls: [UIControl & Shakeable]) { 
	for control in controls where control.state.isEnabled { 
	} 
	control.shake() 
}
```



```swift

```

