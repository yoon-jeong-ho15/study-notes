---
title: 통합 테스트의 한계 - 구매 페이지
date: 2026-02-04
order: 9
---
## 테스트 코드 작성

구매 페이지에는 세 가지 영역이 있다 :
- `<ShippingInformationForm />`
	- 배송 정보 입력
	- API를 통해 쿠폰 정보를 가져오고 렌더링
	- 각 필드별 유효성 검증
- `<ItemList />`
	- 장바구니에 담긴 상품과 가격 정보 노출
- `<Payment />`
	- 총 상품 금액, 할인 쿠폰, 총 결제 금액 등 결제 정보를 계산하여 노출
	- 결제 방법 선택


```jsx
const { resetCart } = useCartStore(state => pick(state, 'resetCart'));
const { user } = useUserStore(state => pick(state, 'user'));
  //...
const methods = useForm({
    defaultValues: {
      name: user?.name ?? '',
      address: '',
      phone: '',
      requests: '',
      coupon: NO_COUPON_ID,
      payment: 'accountTransfer',
    },
  });
const { handleSubmit } = methods;

return (
  <FormProvider {...methods}>
  //...
  <ShippingInformationForm />
  <ItemList />
  <Payment />
  //...
  <Button
     variant="contained"
      size="large"
      onClick={handleClickPurchase}
  >
    구매하기
  </Button>
  </FormProvider>
);
```

### ShippingInformationForm

`ShippingInformationForm`은 각 입력창마다 `useFormContext()`로 `FormProvider`의 컨텍스트와 연동하여 입력값 유효성 검증을 한다. 

```js
const NameTableRow = () => {
  const {
    register,
    formState: { errors },
  } = useFormContext();

  return (
    //...
          <TextField
            {...register('name', { required: '이름을 입력하세요' })}
            type="input"
            error={!!errors?.name}
            helperText={errors?.name?.message}
            size="small"
            placeholder="이름을 입력하세요"
          />
    //...
  );
};
```

그래서 테스트를 진행할 때 이 컨텍스트 프로바이더 컴포넌트도 모킹을 해야한다.

```jsx
//ShippingINformationForm.spec.jsx
const TestForm = props => {
  const methods = useForm({
    defaultValues: {
      name: '',
      address: '',
      phone: '',
      requests: '',
      coupon: NO_COUPON_ID,
      ...props,
    },
  });

  return (
    <FormProvider {...methods}>
      <ShippingInformationForm />
      <button type="button" onClick={methods.handleSubmit(() => {})}>
        테스트 버튼
      </button>
    </FormProvider>
  );
};
```

### ItemList

장바구니에 담긴 상품들의 개수와 총액을 보여주기만 하는 컴포넌트라 검증할게 간단하다. cart 스토어만 모킹해서 상품 개수와 금액이 맞는지만 검증하면 된다.

```jsx
//ItemList.spec.jsx
beforeEach(() => {
  mockUseCartStore({
    cart: {
      6: {
        id: 6,
        title: 'Handmade Cotton Fish',
        price: 100,
        description:
  //...
  
```


### Payment

이 컴포넌트도 `<FormProvider>`의 컨텍스트에 의존하기 때문에 테스트를 진행할 때 프로바이더와 컨텍스트를 포함해 모킹해야 한다.

```jsx
//Payment.spec.jsx
const TestPayment = (props = {}) => {
  const methods = useForm({
    defaultValues: {
      name: 'leejaesung',
      address: '',
      phone: '',
      requests: '',
      coupon: NO_COUPON_ID,
      ...props,
    },
  });

  return (
    <FormProvider {...methods}>
      <Payment />
    </FormProvider>
  );
};
```

## 통합 테스트의 한계

위의 통합테스트를 보면 구매 페이지에 있는 도메인별로 세개의 컴포넌트를 각자 테스트 하고 있다. 각 항목의 입력값 유효성 검증이 잘 이루어지는지, 쿠폰 목록이 제대로 출력되는지, 장바구니의 물품과 총액이 맞게 나오는지 등. 그러나 이 모든 검증을 통과한다는것과 상품 구매 프로세스가 성공적으로 이루어지는것은 다른 문제다.

상품 구매 프로세스에서 결국 가장 중요한 부분은 배송 정보 입력 필드와 결제 금액 정보, 결제 수단 등에 대한 정보를 API에 전달하여 호출했을 때 주문서가 올바르게 만들어지고 이후에 결제 성공과 실패에 따른 흐름이 잘 동작하는지 확인하는 것이다. 

### 구매 페이지 전체를 대상으로 통합 테스트를 작성한다면?

그러기 위해서는 cart 스토어, user 스토어에 대한 모킹과 쿠폰 리스트 api와 구매하기 api 모두에 대한 모킹이 필요하게 된다. 그리고 구매 성공, 실패 여부도 어차피 **모킹에 의존해야 한다**. 

통합 테스트는 이렇게 큰 규모의, 전체 프로세스(워크플로우)를 검증하는데 유효한 테스트가 아니다. 통합 테스트는 **도메인 단위의 비즈니스 로직**과 **컴포넌트간의 상호작용**을 검증하는데 매우 효율적이다. 