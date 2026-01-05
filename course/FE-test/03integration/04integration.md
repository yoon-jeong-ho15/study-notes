---
title: 통합테스트
date: 2025-10-17
order: 1
---
# 1. 통합 테스트란?

두 개 이상의 모듈이 연결된 상태를 검증.
모듈들이 상호 작용햐여 발생하는 상태를 검증.
그래서 실제 앱이 동작하는 비즈니스 로직에 가깝게 기능을 검증할 수 있다.

## 통합 테스트 항목

- 특정 상태를 기준으로 동작하는 컴포넌트 조합
- API와 함께 상호작용 하는 컴포넌트 조합
- 단순 UI 렌더링 및 간단한 로직을 실행하는 컴포넌트(단위 테스트를 하지 않았던)가 제대로 렌더링 되는지 한번에 검증

## 예시

수업 자료의 `ProductList`와 `ProductCard` 컴포넌트를 보자.

```jsx
const ProductCard = ({
  product,
  onClickAddCartButton,
  onClickPurchaseButton,
}) => {
 const navigate = useNavigate();

  if (!product) {
    return null;
  }

  const { title, images, price, category, id } = product;

  const handleClickItem = () => {
    navigate(pathToUrl(pageRoutes.productDetail, { productId: id }));
  };
  
  const handleClickAddCartButton = ev => {
    onClickAddCartButton(ev, product);
  };
  
  const handleClickPurchaseButton = ev => {
    onClickPurchaseButton(ev, product);
  };
  
  return (.....);
};
```

프로덕트 카드는 자신을 호출하는 프로덕트리스트 컴포넌트에서 전달받은 prop들(product 객체, 이벤트 핸들러들)을 그대로 사용한다.
그리고 이 prop들을 가져오고 정의하는 곳은 프로덕트리스트 컴포넌트다.

```jsx
const ProductList = ({ limit = PRODUCT_PAGE_LIMIT }) => {
	...
	...
	
	const products =
    data?.pages.reduce((acc, cur) => [...acc, ...cur.products], []) ?? [];

  const handleClickCart = (ev, product) => {
    ev.stopPropagation();
    if (isLogin) {
      addCartItem(product, user.id, 1);
      toast.success(`${product.title} 장바구니 추가 완료!`, { id: TOAST_ID });
    } else {
      navigate(pageRoutes.login);
    }
  };
  
  const handleClickPurchase = (ev, product) => {
    ev.stopPropagation();
    if (isLogin) {
      addCartItem(product, user.id, 1);
      navigate(pageRoutes.cart);
    } else {
      navigate(pageRoutes.login);
    }
  };
	return (
	    <Grid container spacing={1} rowSpacing={1} justifyContent="center">
	      {products.map((product, index) => (
	        <ProductCard
	          key={`${product.id}_${index}`}
	          product={product}
	          onClickAddCartButton={handleClickCart}
	          onClickPurchaseButton={handleClickPurchase}
	        />
			))}
		...
		...
	);
};
```

먼저 `ProductCard`를 *단위테스트* 했을 때 검증할 수 있는 항목들은 다음과 같다 :
1. **prop 기준**으로 제품 정보가(price, name ..) 제대로 렌더링 되는지
2. 상품을 클릭 했을 때(`handleClickItem`) navigate 모킹을 통해 상세화면으로 이동하는지
3. 장바구니, 구매 버튼을 눌렀을 때 spy 함수를 통해 각 핸들러가 **호출**되는지
	- 호출이 되는지는 검증할 수 있지만 작동하는지 검증할 수 없다.

반면 `ProductList`를 *통합테스트* 했을 때 검증할 수 있는 항목들은 다음과 같다 :
1. 상품 리스트 조회 api에 맞게 상품 정보가 제대로 렌더링 되는지
2. 상품을 클릭 했을 때 navigate 모킹을 통해 상세화면으로 이동하는지
3. 장바구니 혹은 구매 버튼을 눌렀을 때 다음과같이 작동 하는지
	- 로그인 : 상품 추가 후 장바구니로 이동
	- 비로그인 : 로그인 페이지로 이동
4. 상품 리스트가 더 있는 경우 show more 버튼이 노출되고, 이를 통해 데이터를 더 가져올 수 있는지

위의 두 항목들을 비교해보면 1,2,3번은 **중복**된다고 볼 수 있다. (통1,2,3 이 성공한다는것은 단1,2,3 이 성공함을 **함의**한다) 그래서 프로덕트 카드에 대한 단위테스트를 하지 않고 프로덕트리스트의 통합테스트에서 한번에 검증하는것이 더 효과적이다.

# 2. 통합 테스트 대상 선정하기

단위 테스트는 의존성이 적거나 없는 단순한 컴포넌트를 대상으로 한다.
그러나 통합 테스트는 API, 상태 관리 스토어, 리액트 컨텍스트 등 다양한 요소들이 결합된 컴포넌트가 특정 비즈니스 로직을 올바르게 수행하는지 검증. (즉 컴포넌트같의 상호작용, api 호출 및 상태 변경에 따른 UI 변경 사항 검증)

예제속 **메인 페이지**의 *비즈니스 로직*들로는 다음과 같은것들이 있다 :
- 네비게이션 영역의 로그인 여부에 따른 동작
- api를 통해 필터의 카테고리 데이터를 올바르게 렌더링 하는지
- 필터 항목을 수정했을 때 올바르게 반영되는지
- api 응답에 따라 상품 리스트가 적절하게 렌더링 되는지
- 등등

비즈니스 로직 : 
- 프로그램의 핵심 기능을 구현하는 코드
- 즉 사용자가 원하는 결과를 얻기 위한 계산, 처리, 의사 결정을 수행하는 코드

좋은 통합 테스트는 비즈니스 로직을 도메인 단위로 잘 나누어 수행되어야 한다. 
메인 페이지의 비즈니스 로직들은 다음과 같이 분류될 수 있다.
- 네비게이션 바 영역
	- 로그인 여부에 따른 UI 렌더링(장바구니)과 상호작용(로그인, 로그아웃)
- 상품 검색 영역
	- 필터 설정에 따른 검색 조건 설정 
- 상품 리스트 영역
	- 검색 결과에 따른 상품 리스트 렌더링과 버튼 클릭 상호작용(장바구니 추가, 자세히 보기)

만약에 상품 검색 영역에서 검색 필터에 새로운 필드를 추가하거나 하는등의 변경사항이 있거나, 상품 리스트에서 보여주는 상품의 정보가 변경되거나, 스크롤 로딩 방식에서 페이징 방식으로 변경하는 등의 각 도메인에서의 변경사항이 있어도 해당 도메인의 테스트 코드만 변경하면 된다.

즉 다음과 같이 테스트를 설계하면 된다.
1. 메인 페이지를 이루는 모든 요소들을 나열하고, 그 요소들을 도메인 단위로 묶어서 상위 컴포넌트를 만든다.
	- 페이지(`page.tsx`) -> 상위 컴포넌트(`ProductList.tsx`)-> 하위 컴포넌트(버튼, 상품정보 등 렌더링 되는 ui 요소)
2. 그리고 그 상위 컴포넌트에서 자식 컴포넌트들의 상태 관리나 api 호출을 응집시킨다.
3. 통합 테스트는 위의 상위 컴포넌트 단위로 수행한다.

# 3. 상태 관리 모킹하기

*상태* : 시간에 따라 변할 수 있는 데이터. 프론트에서 특히 '상태'가 중요하게 다뤄지는 이유는 UI가 상태에 직접적으로 의존하기 때문이다. 

강의 예제에서 `CartTable` 컴포넌트는 다음과 같은 하위 컴포넌트들로 구성되어 있다.
```jsx
const CartTable = () => {
  return (
    <>
      <PageTitle />
      <ProductInfoTable />
      <Divider sx={{ padding: 2 }} />
      <PriceSummary />
    </>
  );
};
```

여기서 `PageTitle`과 `Divider` 컴포넌트는 너무 단순하기 때문에 테스트가 필요하지 않고

`ProductInfoTable` : 장바구니에 담긴 상품 목록
`PriceSummary` :  가격 총합

위의 두 컴포넌트는 테스트가 필요하다.
어떻게 해야할까? 일단 두 컴포넌트는 서로 의존하고 있지 않고 모두 zustand 스토어에서 상태나 *액션*을 사용한다.
그래서 이 상태와 액션을 모킹한다면 두 컴포넌트 모두 테스트할 수 있게 된다.

주로 루트에 `__mocks__` 폴더를 만들고 하위 경로에 모킹파일을 작성해두면 `vitest`나 `jest` 에서 자동으로 모킹시에 사용한다.

```jsx
// __mocks__/zustand.js
const { create: actualCreate } = await vi.importActual('zustand');
import { act } from '@testing-library/react';

// 앱에 선언된 모든 스토어에 대해 재설정 함수를 저장
const storeResetFns = new Set();

// 스토어를 생성할 때 초기 상태를 가져와 리셋 함수를 생성하고 set에 추가합니다.
export const create = createState => {
  const store = actualCreate(createState);
  const initialState = store.getState();
  storeResetFns.add(() => store.setState(initialState, true));
  return store;
};

// 테스트가 구동되기 전 모든 스토어를 리셋합니다.
beforeEach(() => {
  act(() => storeResetFns.forEach(resetFn => resetFn()));
});
```

```jsx
// src/utils/test/mockZustandStore.jsx
import { useCartStore } from '@/store/cart';
import { useFilterStore } from '@/store/filter';
import { useUserStore } from '@/store/user';

const mockStore = (hook, state) => {
  const initStore = hook.getState();
  hook.setState({ ...initStore, ...state }, true);
};

export const mockUseUserStore = state => {
  mockStore(useUserStore, state);
};

export const mockUseCartStore = state => {
  mockStore(useCartStore, state);
};

export const mockUseFilterStore = state => {
  mockStore(useFilterStore, state);
};

```

주스탠드 모킹 함수들을 보면 바로 주스탠드 스토어의 **상태를 직접 변경**(`hook.setState()`)하는 것을 볼 수 있다.
이게 가능한 이유는 `/__mocks__/zustand.js`에서 테스트 실행시 임시 스토어를 만들기 때문.

### 그런데 어차피 임시 스토어를 만든다면 굳이 모킹 함수 `mockZustandStore.jsx`가 필요할까?

```jsx
export const useUserStore = create(set => ({
  isLogin: Cookies.get('access_token'),
  user: null,
  setIsLogin: isLogin => set(state => ({ ...state, isLogin })),
  setUserData: user => set(state => ({ ...state, user })),
}));
```

원래 주스탠드 액션을 통해 `isLogin`을 변경하려면 `setIsLogin()`을 사용해야 하지만, 위의 방식처럼 직접 상태를 변경하면 다음과 같이 **한번에 여러 상태를 변경**할 수 있다.

```jsx
mockUseUserStore({ isLogin: true, user: { id: 10 } });
// useUserStore.setIsLogin(true);
// useUserStore.setUserData({id:10});
// 위의 두 액션을 동시에 하는 효과
```

#### 스파이 함수 사용을 위해

무엇보다 주스탠드 스토어 모킹 함수가 필요한 이유는 **스파이 함수 사용의 용이성**을 위해서다. 
자세한 내용은 아래 "6. RTL 비동기 유틸 함수를 통한 노출 테스트 작성" 부문에 있다.

# 4. 통합 테스트 작성하기 - 상태 관리 모킹

위에서 말한 두 컴포넌트를 테스트하기 위해 상태 관리를 모킹하는 방법을 알았으니 이제 적용해본다.
## ProductInfoTable

```jsx
const ProductInfoTable = () => {
  const { cart, removeCartItem, changeCartItemCount } = useCartStore(state =>
    pick(state, 'cart', 'removeCartItem', 'changeCartItemCount'),
  );
  const { user } = useUserStore(state => pick(state, 'user'));

  return (
    <TableContainer component={Paper} sx={{ wordBreak: 'break-word' }}>
      <Table aria-label="장바구니 리스트">
        <TableBody>
          {Object.values(cart).map(item => (
            <ProductInfoTableRow
              key={item.id}
              item={item}
              user={user}
              removeCartItem={removeCartItem}
              changeCartItemCount={changeCartItemCount}
            />
          ))}
        </TableBody>
      </Table>
    </TableContainer>
  );
};
```

## 단위 테스트?

하위 컴포넌트인 `ProductInfoTableRow`도 여러 기능을 가지고 있는 컴포넌트인데 더 정확한 테스트를 위해서는 이 컴포넌트먼저 테스트를 해야하지 않을까? 아닐까?

강의에서는 어차피 `ProductInfoTableRow`를 검증할때 할 수 있는건 인자로 전달받은 상태나 함수를 제대로 렌더링하고 호출하는지에 대한 여부 밖에 검증할 수 없으니까 통합테스트에 포함하여, 상품 목록을 제대로 출력하는지를 확인하는게 더 효과적이라고 한다.

여기서 아리송해지는게 그러면 애초에 **단위 테스트가 왜 필요한거였더라?**
유틸함수나 커스텀훅 말고 렌더링 컴포넌트를 단위 테스트로 검증 할 필요가 왜 있는거지?
찾아보니 여러 곳에서 사용되어 복잡한 조건부 렌더링을 하는 컴포넌트에서는 (`<Button variant={} size={} disabled={} onClick={} ...... />`) 단위 테스트를 하는것이 좋고 그 외의 대부분의 렌더링 컴포넌트들은 사실 통합테스트에 겁증한다고 한다.

## 통합 테스트 작성

```jsx
// ProductINfoTable.spec.jsx
beforeEach(() => {
  mockUseUserStore({user: {id: 1}});
  mockUseCartStore({
    cart: {
      6: {
        id: 6,
        title: 'Handmade Cotton Fish',
        price: 809,
        description:
          'The slim & simple Maple Gaming Keyboard from Dev Byte comes with a sleek body and 7- Color RGB LED Back-lighting for smart functionality',
        images: [
          'https://user-images.githubusercontent.com/35371660/230712070-afa23da8-1bda-4cc4-9a59-50a263ee629f.png',
        ],
        count: 3,
      },
      7: {
        id: 7,
        title: 'Awesome Concrete Shirt',
        price: 442,
        description:
          'The Nagasaki Lander is the trademarked name of several series of Nagasaki sport bikes, that started with the 1984 ABC800J',
        images: [
          'https://user-images.githubusercontent.com/35371660/230762100-b119d836-3c5b-4980-9846-b7d32ea4a08f.png',
        ],
        count: 4,
      },
    },
    totalCount: 7,
    totalPrice: 500,
  });
});
```

이렇게 셋업을 통해 상태를 모킹한다.
그리고 다음과 같은 필요한 테스트들을 진행하면 된다.

```jsx
it('장바구니에 포함된 아이템들의 이름, 수량, 합계가 제대로 노출된다', async () => {
  await render(<ProductInfoTable />);

  const [firstItem, secondItem] = screen.getAllByRole('row');

  expect(
    within(firstItem).getByText('Handmade Cotton Fish'),
  ).toBeInTheDocument();
  expect(within(firstItem).getByRole('textbox')).toHaveValue('3');
  expect(within(firstItem).getByText('$2,427.00')).toBeInTheDocument();

  expect(
    within(secondItem).getByText('Awesome Concrete Shirt'),
  ).toBeInTheDocument();
  expect(within(secondItem).getByRole('textbox')).toHaveValue('4');
  expect(within(secondItem).getByText('$1,768.00')).toBeInTheDocument();
});
```

### getBy, queryBy

```jsx
it('특정 아이템의 삭제 버튼을 클릭할 경우 해당 아이템이 사라진다', async () => {
  const { user } = await render(<ProductInfoTable />);
  const [, secondItem] = screen.getAllByRole('row');
  const deleteButton = within(secondItem).getByRole('button');

  expect(screen.getByText('Awesome Concrete Shirt')).toBeInTheDocument();

  await user.click(deleteButton);

  expect(screen.queryByText('Awesome Concrete Shirt')).not.toBeInTheDocument();
});
```

`getByText`는 돔에 해당 텍스트가 존재하는지 확인한다.
그런데 삭제 버튼을 누르면 해당 요소가 사라져야하는데 사라진것을 확인하려면 `getBy`가 아닌 `queryBy` 매처를 사용해야 한다고 한다.

[공식문서](https://testing-library.com/docs/queries/about/)의 설명
- `getBy...`: Returns the matching node for a query, and throw a descriptive error if no elements match _or_ if more than one match is found (use `getAllBy` instead if more than one element is expected).
- `queryBy...`: Returns the matching node for a query, and return `null` if no elements match. This is useful for asserting an element that is not present. Throws an error if more than one match is found (use `queryAllBy` instead if this is OK).
`getBy`는 요소를 찾지 못하면 에러를 발생시키는 반면 `queryBy`는 `null`을 반환한다. 

> 그러면 왜 `queryBy` 로 통일하지 않고 `getBy` 매처도 사용할까?
# 5. msw로 API 모킹하기

Mock Service Worker
강의에서는 너무 간단하게만 설명하고 넘어가서 따로 찾아봤다.[1](https://tech.kakao.com/posts/458) [2](https://velog.io/@khy226/msw%EB%A1%9C-%EB%AA%A8%EC%9D%98-%EC%84%9C%EB%B2%84-%EB%A7%8C%EB%93%A4%EA%B8%B0)

## 기존 모킹 방식

쇼핑몰을 개발중이라면, **제품에 대한 데이터**가 있어야 ui를 구현할 수 있다. 그리고 제품에 대한 데이터는 당연히 서버에 있다. 이 서버 백엔드가 아직 개발되지 않았거나 다른 사유로 인해 사용할 수 없는 경우에는 어떻게 해야할까?

가장 먼저 떠오르는 생각은 그냥 모킹용 데이터를 *앱 안에 직접 작성*하는 것이다. 이렇게는 UI구현까지는 문제 없이 진행할 수 있다.
그런데 서버와 상호작용하는 **api가 제대로 작동하는지도 검증**을 해야한다면?

먼저 *임의의 모킹 서버*를 만들수 있다. 이렇게 하면 직접 서버와 상호작용하는 서비스 로직을 변경할 필요 없이 그대로 사용할 수 있다. 하지만 이 방법은 비용(시간, 인력 등)이 많이 소모되는 방법이라 선호되지 않는다.
그리고 모킹 서버의 단점은 **예외 처리에 대한 테스트가 어렵다**는 단점이 있다.
오류가 발생했고, 어떤 오류인지, 로딩이 길어지면 어떻게 해야하는지 등등 사용자 경험과 관련된 여러 ui와 로직을 개발하려면 개발자가 **쉽게 서버 응답 상태를 바꿀 수 있어야**하는데, 모킹 서버 사용시에는 서버의 코드를 바꾸거나 혹은 기타 다른 방법을 사용해야 한다.

## MSW

MSW 라이브러리는 이런 단점을 손쉽게 해결해준다.
이 라이브러리의 역할은, 브라우저가 전송하는 api 요청을 가로채서 모킹된 응답을 브라우저에게 전달한다.
브라우저를 떠난 뒤에 응답이 오는것이기 때문에 서버를 사용하는것과 동일한 방식으로 테스트를 진행할 수 있다.
그리고 손쉽게 응답의 상태를 변경할 수 있어 예외 처리 로직을 개발할 때 용이하다.

이 라이브러리는 브라우저의 *Service Worker*라는 기술을 활용해 이런 편리를 제공하고 있다.
일단 *Worker*라는 것이 있는데, 이것은 간단히 브라우저가 웹 앱을 실행하고 있는 메인 스레드가 아닌 백그라운드 스레드에서 자바스크립트 문서를 실행할 수 있다.
그리고 Service Worker는 앱과 브라우저 그리고 네트워크 사이에서 프록시 서버 역할을 하는 Worker를 말한다.

> 구체적인 사용법은 다음 강의에서 다룬다.
# 6. RTL 비동기 유틸 함수를 통한 노출 테스트 작성

이제 `ProductList` 컴포넌트에서 **데이터를 가져오는 api**(`useProducts`)가 제대로 작동해서 화면에 올바르게 렌더링 되는지 확인하기 위한 코드를 작성한다.

```jsx
it('로딩이 완료된 경우 상품 리스트가 제대로 모두 노출된다', async () => {
  await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

  const productCards = screen.getByTestId('product-card'); // 사전에 정의함

  expect(productCards).toHaveLength(PRODUCT_PAGE_LIMIT);

  productCards.forEach((el, index) => {
    const productCard = within(el);
    const product = data.products[index];

    expect(productCard.getByText(product.title)).toBeInTheDocument();
    expect(productCard.getByText(product.category.name)).toBeInTheDocument();
    expect(
      productCard.getByText(formatPrice(product.price)),
    ).toBeInTheDocument();
    expect(
      productCard.getByRole('button', { name: '장바구니' }),
    ).toBeInTheDocument();
    expect(
      productCard.getByRole('button', { name: '구매' }),
    ).toBeInTheDocument();
  });
});
```

`getByTestId` : 프로덕트카드 컴포넌트는 지금까지 사용해온 매처 `getByText`나 `getByRole`를 통해 확인하기 힘들기 때문에 미리 `ProductCard`컴포넌트의 테스트 아이디를 지정해서 그것으로 찾을 수 있다.
	- 컴포넌트의 속성에 이렇게 추가해서 정의했다 - `data-testid="product-card"` 

그런데 위의 테스트에는 문제가 있는데, `getByTestId` 를 사용하는 것이다.
테스트 코드는 기본적으로 동기적으로 실행되어 비동기(Promise) 코드는 실행하지 않는다.
그러나 상품 목록을 가져오는 API 호출은 비동기 방식으로 작동하고 promise를 반환하기 때문에 `getBy`로 렌더링 결과를 확인할 수 없게 되는것이다.
대신에 `findBy...` 함수를 사용하면 정해진 시간동안 여러번 재시도를 반복해 api 응답이 완료되어 완전히 데이터를 가져올 때 까지 기다렸다가 성공 여부를 확인할 수 있다.

## 통합테스트에서의 스파이 함수

단위 테스트에서 스파이 함수를 사용해 **함수의 호출 여부와 전달되는 인자들을 검증**할 수 있었다. 
통합 테스트에서도 스파이 함수를 사용하는데 수업에서 사용한 방식은 다음과 같다.

```jsx
it('장바구니 버튼 클릭시 "장바구니 추가 완료!" toast를 노출하며, addCartItem 메서드가 호출된다.', async () => {
    const addCartItemFn = vi.fn();
    mockUseCartStore({ addCartItem: addCartItemFn });

    const { user } = await render(<ProductList limit={PRODUCT_PAGE_LIMIT} />);

    await screen.findAllByTestId('product-card');

    // 첫번째 상품을 대상으로 검증한다.
    const productIndex = 0;
    const product = data.products[productIndex];
    await user.click(
      screen.getAllByRole('button', { name: '장바구니' })[productIndex],
    );

    expect(addCartItemFn).toHaveBeenNthCalledWith(1, product, 10, 1);
    expect(
      screen.getByText(`${product.title} 장바구니 추가 완료!`),
    ).toBeInTheDocument();
  });
});
```

`addCartItem`이라는 스토어 액션을 스파이함수로 대체하고 있다. `src/utils/test/mockZustandStore.jsx` 에 주스탠드 모킹 함수를 작성해 두었는데

```Jsx
// ProductList.jsx
const { addCartItem } = useCartStore(state => pick(state, 'addCartItem'));
...

const handleClickCart = (ev, product) => {
	ev.stopPropagation();
	if (isLogin) {
	  addCartItem(product, user.id, 1);
	  toast.success(`${product.title} 장바구니 추가 완료!`, { id: TOAST_ID });
	} else {
	  navigate(pageRoutes.login);
	}
};
```

테스트용으로 렌더링 된 컴포넌트는 저 handleClickCart가 실행될 때  주스탠드에 정의되어 있는 기존의 `addCartItem`이 아닌 스파이 함수를 호출한다.
