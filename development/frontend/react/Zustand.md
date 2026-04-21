## Doc
### create Zustand
```
它是一个Zustand库的核心方法，它创建了一个全局状态管理的hook，这个数据的内容是全局共享的，普通hook的作用域不是全局共享的
它创建的只是一个普通的js对象，不是interface的实例，意思是创建的对象，要符合AuthState的结构

interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  _hydrated: boolean;
  setUser: (user: User | null) => void;
  setToken: (token: string | null) => void;
  login: (token: string) => Promise<void>;
  logout: () => Promise<void>;
  refreshUser: () => Promise<void>;
}

export const useAuthStore = create<AuthState>()
```
### persist
```
将对象存储在localstorage。

	{
      name: "viralcopy-auth",
      partialize: (state) => ({ token: state.token }),
      onRehydrateStorage: () => (state) => {
        if (state) state._hydrated = true;
      },
    }
localStorage的名字: 
viralcopy-auth

partialize: (state) => ({ token: state.token })
只存token，别的都不存

onRehydrateStorage: () => (state) => { ... }
页面刷新后，persist自动把token读回来，

```
### set
```
(set) => { ... }
修改全局状态的数据。所以set必须用在create里面，设置_hydrated，这样页面就知道恢复完成了，可以用了。
```
### useAuthStore
```
const { user, token, isLoading, login, logout, refreshUser } = useAuthStore();
const user = useAuthStore((s) => s.user);

if you pass only a function, you only get that piece of state
if you pass nothing, you get the entire store object.
```
## sample
```
interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  _hydrated: boolean;
  setUser: (user: User | null) => void;
  setToken: (token: string | null) => void;
  login: (token: string) => Promise<void>;
  logout: () => Promise<void>;
  refreshUser: () => Promise<void>;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isLoading: false,
      _hydrated: false,

      setUser: (user) => set({ user }),
      setToken: (token) => set({ token }),

      login: async (token: string) => {
        set({ isLoading: true });
        localStorage.setItem("access_token", token);
        set({ token });
        try {
          const user = await userApi.getMe();
          set({ user });
        } finally {
          set({ isLoading: false });
        }
      },

      logout: async () => {
        try { await authApi.logout(); } catch { /* ignore */ }
        localStorage.removeItem("access_token");
        set({ user: null, token: null });
      },

      refreshUser: async () => {
        const token = localStorage.getItem("access_token");
        if (!token) return;
        try {
          const user = await userApi.getMe();
          set({ user });
        } catch {
          localStorage.removeItem("access_token");
          set({ user: null, token: null });
        }
      },
    }),
    {
      name: "viralcopy-auth",
      partialize: (state) => ({ token: state.token }),
      onRehydrateStorage: () => (state) => {
        if (state) state._hydrated = true;
      },
    }
  )
);
```
