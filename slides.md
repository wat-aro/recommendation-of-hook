# `React Hooks` のすすめ

***

## `About me`

- wat-aro
- @wat_aro
- Github: wat-aro

---

### お仕事関係

- 元陸上自衛官
- Fjord卒業生
- 永和システムマネジメントに入社して5年目になりました

---

### 好きなもの

- 関数型言語(Haskell, Elm, Scheme, Coq)
- 筋トレはじめました
- 最近仕事でReactを書いていて気に入ったReact Hooksを紹介します

***

***

## `React Hooks` とは

- React 16.8 で追加された機能。
- Functional Componentにstateや副作用をもたせることができる

---

### `React Hooks` 以前

- Functional Component は Presentational Component として使う
- stateや副作用が必要な場合はClassコンポーネントやHOCを使います

---

### `React Hooks` 以前

```jsx
class Users extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      company: props.company,
      users: [],
    };
  }

  async componentDidMount() {
    const usersResponse = await Axios.get<User[]>('/users');
    this.setState({ ...this.state, users: usersResponse.data });
  }

  render() {
    return (
      <div>
        <h3>{this.state.company.name}</h3>
        <ul>
          {this.state.users.map((user) => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      </div>
    );
  }
}
```

***

***

## `Ract Hooks` でどう変わるか

``` jsx
const Users: React.FC<Props> = (props) => {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    (async () => {
      const usersResponse = await Axios.get<User[]>('/users');
      setUsers(usersResponse.data);
    })();
  }, []);

  return (
    <div>
      <h3>{props.company.name}</h3>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

---

### `useState`

```typescript
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>];
function useState<S = undefined>(): [S | undefined, Dispatch<SetStateAction<S | undefined>>];
```

- useStateに値を渡すと、それを初期値とするデータとデータを更新する関数のタプルを返す。
- データを更新する関数を使うとデータが更新され、再レンダーされる

```typescript
const [state, setState] = useState<T>(initialValue);
```

---

### `useEffect`

- レンダー後に実行される(≒ componentDidMount)
- 第二引数に依存する変数の配列を書く
- 第二引数を設定しない場合はレンダーされるたびに実行される(≒ componentDidUpdate)
- 第二引数に依存する変数を指定すると、その変数が変更された場合のみ実行される
- 同じコンポーネント内で複数回使える
- useEffectから関数を返すとクリーンアップ用関数として実行される(≒ componentWillUnmount)

---

### `useEffect`

- 以前はライフサイクルメソッドしかなかったため複数の関心事が同じメソッド内で実行されていた。
- useEffectは複数回使えるため関心事を分離できる

---

### `useEffect`

別々の関心事をそれぞれのuseEffectで実行している

``` typescript
  useEffect(() => {
    (async () => {
      const usersResponse = await Axios.get<User[]>('/users');
      setUsers(usersResponse.data);
    })();
  }, []);

  useEffect(() => {
    const subscription = props.source.subscribe();
    return () => {
      subscription.unsubscribe();
    };
  });
```

---

### 関心事をカスタムフックに抽出する

users の取得部分をカスタムフックに切り出してみる

``` typescript
export const useUsers = () => {
  const [users, setUsers] = useState<User[]>([]);

  useEffect(() => {
    (async () => {
      const usersResponse = await Axios.get<User[]>('/users');
      setUsers(usersResponse.data);
    })();
  }, []);

  return { users };
};
```

---

### カスタムフックを使う

props で受け取るのと同様に扱える

``` jsx
const Users: React.FC<P> = (props) => {
  const { users } = useUsers();

  return (
    <div>
      <h3>{props.company.name}</h3>
      <ul>
        {users.map((user) => (
          <li key={user.id}>{user.name}</li>
        ))}
      </ul>
    </div>
  );
};
```

---

### カスタムフックを拡張する

- ロード中の状態もほしい
- エラーがある場合はそれもほしい

---

### カスタムフックを拡張する

- 状態と初期値を定義する

``` typescript
type State = {
  users: User[];
  isLoading: boolean;
  error?: Error;
};

const initialState = {
  users: [],
  isLoading: false,
};
```

---

### カスタムフックを拡張する

よしなにデータを更新してあげる

``` typescript
export const useUsers = () => {
  const [data, setState] = useState<State>(initialState);

  useEffect(() => {
    (async () => {
      setState({ ...data, isLoading: true });
      try {
        const usersResponse = await Axios.get<User[]>('/users');
        setState({ ...data, isLoading: false, users: usersResponse.data });
      } catch (error) {
        setState({ ...data, isLoading: false, error: error });
      }
    })();
  }, []);

  return { ...data };
};
```

---

### カスタムフックを拡張する

コンポーネントに組み込むと、ロジックはカスタムフックにあって、コンポーネントは状態に合わせて表示するだけになる

``` jsx
const Users: React.FC<P> = (props) => {
  const { users, isLoading, error } = useUsers();

  return (
    <div>
      <h3>{props.company.name}</h3>
      {isLoading ? (
        'Loading ...'
      ) : error !== undefined ? (
        <Error error={error} />
      ) : (
        <ul>
          {users.map((user) => (
            <li key={user.id}>{user.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
};

```

---

### 他の Hook

よく使うもののみ

- useContext
  - グローバルな状態を扱える
- useReducer
  - Redux みたいなやつ
- useMemo
  - コンポーネント内で定義する変数をメモ化できる
- useCallback
  - 関数をmemo化するためのhook

***

***

## 実際使っての感想

---

### pros

- useStateやuseReducerから返される値が変化するだけなのでより宣言的に書けてよい
- class を使わないので this も使うことがなくなってよい
- useContext と useReducerで実質Reduxみたいなこともできる
- ロジックはhooksに寄せて単体テスト、見た目はRegression Testの体験がよい
- TypeScriptとの相性がよい

---

### cons

- useEffect で気をつけないと無限ループ
- 配列でない値が `T | undefined` になりがち(hooks に限ったことではない)

---

### 最近のなやみごと

Hooks によって Container Component がいらなくなると思っていたけれど、useStateやuseEffectは結局副作用なのでPresentational ComponentとContainer Componentに分けたほうがいいのではという気持ちになってきた

---

### おわりに

Algebraic Effectとの関係などを調べて発表したかったのですが、力及ばず。
Promise を throw するで話題になった Concurrent mode も気になってます。
