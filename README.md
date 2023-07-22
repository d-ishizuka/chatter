# chatter

# API設定
## 7
- frontディレクトリの作成
 - npx create-next-app client --ts
- backendディレクトリの作成(api)
 - npm init -y
 - package.jsonの変更
 - server.jsの作成

## 8
- experssとのnodemonをインストール
- server.jsでexpressサーバーを立てる
 - app.listen
 - npm run dev(package.jsonに設定済)
 - getのエンドポイントを試しに作ってみる

## 9
- データベースの用意
 - supabaseにログイン
  - 簡単に設定できるので使う(backend as a service)
 - new projectを作成
  - DB用のパスワード設定する
  - postgresにする

## 10
- prisma
 - ORMとして機能する
 - サイトに行って、create project setupに従って設定
 - npx prisma initするとディレクリが作成されてschemaファイルができる
 - .envにsupabaseに書いたパスワードなどを設定する

## 11
- モデルを作成する
 - UserモデルとPostモデルを作成する
  - autoincrementでid自動で振れる
  - @uniqueでユニークキー
  - []で配列指定 
  - @relation
 - migrationする
  - sql文に生成する
  - supabaseに繋がってないとだめ

## 12
- ユーザー登録のAPI作成
    - app.post(path, ...)
    - req.bodyで受け取る
    - prisma clientインストールしてimportする
    - create, findManyとかprismaの作法がある(ORMなので)
    - schema prismaに設定されているモデルの小文字が呼び出せる(User→user)
        - ActiveRecordのようにルールがある

## 13
- パスワードの暗号化
    - bcryptをインストールしてインポート
    - bcryptでハッシュ化
        - 使い方はドキュメント参照

## 14
- Thunder Clientを使うとpostman的なことができる
- express側でjson形式であることを設定する
- supabaseにデータを保存されている

## 15
- ユーザーログインのAPIを作成
    - Tokenを発行するAPI(/api/auth/login)
    - email, passwordを受け取ってuserを検索
    - ユーザー、パスワード見つからなかったらエラーを返す

## 16
- JWT(JSON WEB TOKEN)を発行して返す
    - jsonwebtokenをインストールしてimport
    - user idをもとに、jwtを発行
        - expireの起源とかも決める
        - SECRET_KEYはかなり長いものを指定するのが一般的
        - .envから呼び出してgitに上げない
    - SECRET_KEYを.envから呼び出すために、dotenvをインストールしてrequire

## 17
- Thunder Clientで投げてみる
- jwtのサイトいくと、発行されたjwtをデコードできる

## 18
- apiファイルを分割する
 - routersディレクトリを作成してserver.jsに書いてた各種エンドポイントを分割する
  - auth.js, user.js...
  - 各種ファイルでrouterをrequireで呼び出して、module.exports = routerでエクスポート
  - server.js側で各routerを呼び出す(app.useでルートと、authRouteとかを結びつける)
  - Thunder Clientで正しいか確認する

# Front設定
## 19
- tailwindのセットアップをする
    - next.js tailwindcdでググる
    - npm install -D tailwindcss postcss autoprefixer
    - tailwind.config.jsを修正する
    - global.cssを修正する
    - index.tssでなんか適当にスタイル当てて効いてるか確認

## 20
- sns用のナビゲーションバーを作成する
    - componetnsの中に、Navbar.tsxを作成する
        - これはどのページからも呼び出されるコンポーネント
        - _app.tsxで読んでおけば、どこのページでも呼び出せる
            - application.html.erbと同じようなノリ

## 21
- 投稿用のタイムラインコンポーネントを作成する
    - components/Timeline.tsx
    - index.tsにインポートする
    - Post.tsxを一旦ハードコーディングで作成する
    - Timeline.tsxのcomponentsの中で呼び出すことで、擬似的なタイムラインが表示できる
    - この後、Post.tsxをループでDBからとって表示できるようにする

## 22
- 新規登録フォームとログインフォームを作成する
    - loginページを作りたいので、api/login.tsxを作る
    - global.cssをいじると、サイトの背景色とかも一括で変更できる
    - signupページを作りたいので、api/signup.tsxを作る

## 23, 24
- フォームに入力した値を取得する
    - useStateで入力された値を取ってくる
        - e.g. const [name, setName] = useState<string>("");
        - e.g. フォームのinputタグの中で、onChange={(e) => setName(e.target.value)}
    - formタグのところにonSubmit={() => handleSubmit}を設定する
        - const handleSubmit = (e: any) => {e.preventDefault(); console.log(name)}とかすると、submitのイベントを中断して、送信しようとしている値を確認することができる
        - preventして、この値をapiにとくれば良い
            - 送信するために、axiosをインポートする(fetch関数でもいい)
            - lib/apiClient.tsファイルを作成する
                - axiosをimportして、axios.createする
                    - baseUrlとか設定する
                    - headerでcontentTypeとかも設定する
            - signup.tsxでこの関数を呼んで実行する
                - pathはbaseUrlの以降を指定
                - apiの時に、server.jsでroute設定しているのでそれに合わせればいい
        - 成功したら、リダイレクトする
            - route.push(リダイレクト先)で設定できる
            - 具体的には、ユーザー登録したらログインページに遷移させる
## 25
- corsのエラーを解決する
    - localhost:3000 → localhost:5000へのアクセスを許可する
    - apiがのserver.js側でcorsをインストールして、requireしてuseすることでcorsの許可ができる
        - 実際にはちゃんとアクセス元を絞る必要がある
    - 今回で言うと、auth.jsでリクエストを処理するが、postリクエストする時のattributesは送信側と受け取る側で一致してないとダメなので注意

## 26
- Next.js側からログインAPIを叩く
    - 基本、signup.tsxを参考に修正
    - login.tsxから登録済みのユーザーネーム・emailを送信するとjwtがresponseとして返ってくる
        - response.data.tokenでアクセスできる
    - クライアント側は、response.data.tokenの中身をローカルストレージに保存する
        - この処理はどこの処理でも使う可能性があるので、context/auth.tsxで定義しておく

## 27, 28
- 受け取ったjwtトークンをローカルストレージに保存する
 - context/auth.tsxを定義する
    - React.createContextでフックイベントを作ることができ、その関数の中に処理を書く
    - useAuthみたいな関数を定義して、作成したフックイベントをuseContext(AuthContext)みたいに渡すとフックしてこの処理を外から呼べる
    - Proveider関数を定義する
        - 引数はchiledren(詳細は調べる)
        - login関数を中に定義する
            - tokenを受け取っとたら、localStrage.setItem("auth_token", token)みたいにする
        - logout関数を中に定義する
            - localStorage.removeItem("auth_token")
        - const valueの中に、二つの関数をハッシュで入れる
        - returnで<AuthContext.Provider>で囲む
            - {children}の子コンポーネントで、常にvalueが使えるようになる
        - 要するにアプリケーション全体で使いたいものについては、Reactのコンテキストを使うと使えるようになるよという話
            - ログアウトとかどこからでもできるようなイメージ
            - そこまで複雑ではないので、reduxとか使わずuseContextを使った
    - 型を登録する
        - chiledren: ReactNode
        - 関数は、(token: stirng) => voidとか

## 29
- アプリケーション全体でAuthProviderを使う
    - _app.tsxでNavbarとか全体を、<AuthProvider>で囲むこと
        - この時、AuthProveider関数で定義したchildrenは囲まれたNavbarとかpagePropsのcomponent
        - つまり、囲ったこのcomponentの中では、AuthProviderが使えるということ(valueが使える)
    - よく使うものをいちいち親コンポーネントからバケツリレーすると大変なことになるので、こういう書き方する
    - 例えば、login.tsx関数で、useAuth()の返り値(userContextの生成物)を受けることができる

# つぶやき投稿のAPIと最新のつぶやきを取得するAPIを作る
## 30
- api/routers/posts.jsをauth.jsをベースに変更する
    - つぶやきの内容を受け取る→なければエラー
    - await prisma.post.create()でプリズマのORMを使う
    - res.status(xxx).json(yyy)でresponse設定できる
- 