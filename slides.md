# `React Hooks` のすすめ

***

### `About me`

- wat-aro
- @wat_aro
- Github: wat-aro

---

#### お仕事関係

- 元陸上自衛官
- Fjord卒業生
- 永和システムマネジメントに入社して5年目になりました

---

#### 好きなもの

- 関数型言語(Haskell, Elm, Scheme, Coq)
- 筋トレはじめました
- 最近仕事でReactを書いていて気に入ったReact Hooksを紹介します

***

***

### `React Hooks` とは

React 16.8 で追加された機能。
Functional Componentにstateや副作用をもたせることができる

---

#### `React Hooks` 以前

- Functional Component は Presentational Component として使う
- stateや副作用が必要な場合はClassコンポーネント
- コンポーネント間で共通の処理をまとめたい場合はHOC

---

#### `React Hooks` 以前

```typescript
class Users extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
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
        <h3>{this.props.company.name}</h3>
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

### `Ract Hooks` でどう変わるか

``` typescript
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

#### `useState`

``` typescript
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>];
function useState<S = undefined>(): [S | undefined, Dispatch<SetStateAction<S | undefined>>];
```

- useStateに値を渡すと、それを初期値とするデータとデータを更新する関数のタプルを返す。
- データを更新する関数を使うとデータが更新され、再レンダーされる

``` typescript
const [data, setData] = useState<T>(initialValue);
```

---

#### `useEffect`

- レンダー後に実行される(≒ componentDidMount)
- 第二引数を設定しない場合はレンダーされるたびに実行される(≒ componentDidUpdate)
- 第二引数に依存する変数を指定すると、その変数が変更された場合のみ実行される
- componentDidmount などのライフサイクルメソッドの代わりになる
- 同じコンポーネント内で複数使える
- useEffectから関数を返すとクリーンアップ用関数として実行される(≒ componentWillUnmount)

---

#### `useEffect`

- 以前はライフサイクルメソッドしかなかったため複数の関心事が同じメソッド内で実行されていた。
- useEffectを使うことで関心事を分離できる
- 
