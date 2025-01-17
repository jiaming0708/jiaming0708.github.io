---
title: '[Angular] 自訂Validator-password format、repeat'
date: 2018-02-17 23:08:09
updated: 2018-02-17 23:08:09
categories:
- Frontend
- Angular
tags:
- Angular
---

# 前言

在寫Angular專案時，只要遇到表單的部份，通常都會使用驗證的機制來輔助使用者輸入，並且提示訊息，Angular團隊也已經寫好一些驗證的pattern讓我們使用，但總是會有些特殊的格式，勢必要自己寫，這邊來跟大家分享一下作法。

<!--more-->

> 底下範例將採用reactive form的方式來介紹

# 範例

## 密碼格式

最簡單的寫法是直接在formControl後面加上pattern，如果只有一個用到，那這樣寫其實就可以，但如果說有重複用到這樣就不適合。

```typescript
ngOnInit(){
  this.form = this.fb.group({
      password: ['', [Validators.required, Validators.pattern('^(?=.*[0-9])(?=.*[a-zA-Z]).{6,}$')]]
  });
}
```

因此我們可以將程式抽出來變成一個function，然後呼叫使用。

這邊要注意一下，當驗證成功是要回傳null，驗證失敗時可以回傳一個物件，這個物件的內容自訂，像是底下的範例，是給`password=true`這可以用在html決定是否提示錯誤的內容。

```typescript
ngOnInit(){
  this.form = this.fb.group({
      password: ['', [Validators.required, this.password]]
  });
}

password(control: AbstractControl): ValidationErrors | null {
	const pattern = /^(?=.*[0-9])(?=.*[a-zA-Z]).{6,}$/;
	return pattern.test(control.value) ? null : { 'password': true };
}
```

```html
<form [formGroup]="form">
  <input type="password" formControlName="password">
  <span *ngIf="form.get('password').errors?.password">密碼必須六位數且包含英數字。</span>
</form>
```

## 確認密碼檢查

這邊提供兩種方式，第一種是在formGroup，第二種是在formControl

> 個人偏好第一種，寫法乾淨

* 在formGroup加上validator，並且傳入兩個controlName，作為判別

```typescript
export class CustomValidator{
    static match(firstControlName: string, secondControlName: string) {
        return (control: AbstractControl): { [key: string]: any } => {
            const firstControl = control.get(firstControlName);
            const secondControl = control.get(secondControlName);

            if (firstControl.value !== secondControl.value) {
                secondControl.setErrors({ 'match': true });
            } else {
                return null;
            }
        };
    }
}

this.resetForm = this.fb.group({
	password: ['', Validators.required],
	confirmPassword: ['', Validators.required]
}, {validator: CustomValidator.match('password', 'confirmPassword')});
```

> 這邊使用另一種寫法，回傳`ValidatorFn`，有別於直接回傳驗證結果，是為了要能夠傳參數進去

* 在重複輸入formControl上，傳入比對的controlName，這邊處理上比較麻煩一點，我們要先假設一下情境
  1. 正常情況下，先輸入password，然後再輸入confirmPassword，這時的驗證就是看自己和比對有沒有一致就好。
  2. 但可能會是兩邊都輸入完而且驗證也過了，但去改了password，這時的驗證應該要重新跑過，因此我們需要監聽比對的值變更來讓自己的驗證重新執行，因此要用到`updateValueAndValidity`這個功能

```typescript
export class CustomValidator{
    static repeat(controlName: string): ValidatorFn {
        let otherControl: AbstractControl;
        return (control: AbstractControl): { [key: string]: any } => {
            if (!control.parent) {
                return null;
            }

            if (!otherControl) {
                otherControl = control.parent.get(controlName);
                otherControl.valueChanges.subscribe(p => control.updateValueAndValidity());
            }

            return otherControl.value !== control.value ? { 'match': true } : null;
        };
    }
}

this.form = this.fb.group({
	password: ['', Validators.required],
	confirmPassword: ['', [Validators.required, CustomValidator.repeat('password')]]
});
```

# 參考

* [官網介紹](https://angular.io/guide/form-validation)
* [CUSTOM VALIDATORS IN ANGULAR](https://blog.thoughtram.io/angular/2016/03/14/custom-validators-in-angular-2.html)
* [Advanced Validation with Angular Reactive Forms](https://medium.com/@amcdnl/advanced-validation-with-angular-reactive-forms-2929759bf6e3)

