---
title: Phoenix LiveView 介绍
date: 2025-05-11 11:00:00
tags: [Elixir, Phoenix, LiveView]
categories: [Web Backend]
---

# 什么是 LiveView？

[Phoenix LiveView](https://hexdocs.pm/phoenix_live_view/welcome.html) 是 Elixir/Phoenix 框架下的一种模式，用来构建无需 JavaScript 的高交互网页。其主要特点包括：

- 运行在 **服务器端的组件化 UI**
- 通过 **WebSocket** 实现 **页面局部更新**
- 类似于 React 的状态驱动渲染，但渲染逻辑在服务器端完成
- **零 JS 也能做出动态交互**：表单验证、实时列表等

LiveView 把我们过去需要写 JS 和前后端通信才能做的事，变成纯后端代码就能完成的工作

# LiveView 应用演示

## 任务列表

<iframe width="100%" height="624" src="https://www.youtube.com/embed/Onb55RSUolI" title="Phoenix LiveView Todo Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

```elixir
defmodule LiveViewDemoWeb.TodoLive do
  use LiveViewDemoWeb, :live_view
  import Ecto.Query

  alias LiveViewDemo.Task
  alias LiveViewDemo.Repo

  @topic "todo"

  def render(assigns) do
    ~H"""
      <div class="max-w-md mx-auto mt-10 p-4 border rounded">
        <h2 class="text-2xl font-bold mb-4">Todo List</h2>

        <form phx-submit="add">
          <input type="text" name="title" value={@title}
                class="border p-2 w-full mb-2" placeholder="New task..." />
          <button class="bg-blue-500 text-white px-4 py-1 rounded">Add</button>
        </form>

        <ul class="mt-4 space-y-2">
          <%= for task <- @tasks do %>
            <li class="flex justify-between items-center border p-2 rounded">
              <div class="flex items-center space-x-2">
                <input type="checkbox"
                      phx-click="toggle"
                      phx-value-id={task.id}
                      checked={task.completed} />
                <span class={if task.completed, do: "line-through text-gray-500", else: ""}>
                  <%= task.title %>
                </span>
              </div>
              <button phx-click="delete" phx-value-id={task.id}
                      class="text-red-500 hover:underline">Delete</button>
            </li>
          <% end %>
        </ul>
      </div>
    """
  end

  def mount(_params, _session, socket) do
    Phoenix.PubSub.subscribe(LiveViewDemo.PubSub, @topic)

    {:ok, assign(socket, tasks: list_tasks(), title: "")}
  end

  def handle_event("add", %{"title" => title}, socket) do
    if String.trim(title) != "" do
      %Task{title: title, completed: false}
      |> Repo.insert()
      Phoenix.PubSub.broadcast(LiveViewDemo.PubSub, @topic, :tasks_updated)
    end
    {:noreply, assign(socket, tasks: list_tasks(), title: "")}
  end

  def handle_event("delete", %{"id" => id}, socket) do
    task = Repo.get!(Task, id)
    Repo.delete!(task)
    Phoenix.PubSub.broadcast(LiveViewDemo.PubSub, @topic, :tasks_updated)
    {:noreply, assign(socket, tasks: list_tasks())}
  end

  def handle_event("toggle", %{"id" => id}, socket) do
    task = Repo.get!(Task, id)
    {:ok, _} =
      task
      |> Ecto.Changeset.change(completed: !task.completed)
      |> Repo.update()

    Phoenix.PubSub.broadcast(LiveViewDemo.PubSub, @topic, :tasks_updated)
    {:noreply, assign(socket, tasks: list_tasks())}
  end

  def handle_info(:tasks_updated, socket) do
    {:noreply, assign(socket, tasks: list_tasks())}
  end

  defp list_tasks do
    Repo.all(from t in Task, order_by: [desc: t.inserted_at])
  end
end
```

## 用户注册表单

<iframe width="100%" height="624" src="https://www.youtube.com/embed/uxG8sW_IG_4" title="Phoenix LiveView Registration Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

```elixir
defmodule LiveViewDemoWeb.RegistrationLive do
  use LiveViewDemoWeb, :live_view

  alias LiveViewDemo.Accounts.User
  alias LiveViewDemo.Repo

  def render(assigns) do
    ~H"""
      <.simple_form
        for={@changeset}
        as={:user}
        id="registration_form"
        phx-change="validate"
        phx-submit="save"
        class="space-y-6"
        :let={f}
      >
        <.input field={f[:username]} label="Username" />
        <.input field={f[:password]} type="password" label="Password" />

        <:actions>
          <.button phx-disable-with="Registering..." class="w-full">
            Register
          </.button>
        </:actions>
      </.simple_form>
    """
  end

  def mount(_params, _session, socket) do
    {:ok, assign(socket, changeset: User.changeset(%User{}, %{}))}
  end

  def handle_event("validate", %{"user" => user_params}, socket) do
    changeset =
      %User{}
      |> User.changeset(user_params)
      |> Map.put(:action, :validate)

    {:noreply, assign(socket, changeset: changeset)}
  end

  def handle_event("save", %{"user" => user_params}, socket) do
    case Repo.insert(User.changeset(%User{}, user_params)) do
      {:ok, _user} ->
        socket = put_flash(socket, :info, "Registration successful!")
        {:noreply, push_navigate(socket, to: "/")}
      {:error, changeset} ->
        {:noreply, assign(socket, changeset: changeset)}
    end
  end
end
```

## 其他

https://github.com/jinhucheung/live_view_demo/tree/main/lib/live_view_demo_web/live

<iframe width="100%" height="831" src="https://www.youtube.com/embed/l7oWwYKkXP8" title="Phoenix LiveView Chat Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="100%" height="624" src="https://www.youtube.com/embed/MH_Pvo33ArE" title="Phoenix LiveView Counter Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="100%" height="624" src="https://www.youtube.com/embed/__02rnWvYuM" title="Phoenix LiveView Weather Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# LiveView 要解决哪些痛点？

对于大部分传统 MVC 后端框架，如果要实现：

- 实时表单验证
- 动态页面交互

必须写 JavaScript：

- 用 JS 监听事件
- 用 AJAX 发送请求
- 写后端接口返回 JSON
- JS 再根据数据更新 DOM

在这个过程中会有下面问题：

- **前后端割裂**
- **状态同步困难**
- **维护成本高**
- **动态交互门槛高**

LiveView 旨在解决传统后端框架中遇到的上述问题，提升实时交互应用开发的体验，减少开发和维护成本

![](/images/posts/26_phoenix_liveview_intro/liveview_1.png)

# LiveView 是否解决了这些问题？

LiveView 优势：

- 满足大部分实时交互需求：实时性强，WebSocket 提供流畅体验
- 高效的开发体验：只需要开发维护一套后端代码，状态完全由后端控制
- SEO 友好：首屏加载返回 HTML
- 出色的性能：仅传输最小化 diff patches 减少了带宽和延迟；基于 Erlang/Elixir 每个用户连接都是独立轻量的进程，天生适合处理高并发；不需要引入前端框架，降低了前端渲染的开销

LiveView 劣势：

- 无法离线使用，不适合复杂的客户端应用(如 Figma)
- 高频交互（如拖拽、动画） 场景下，由于所有事件都需往返服务器，可能不如纯前端流畅

# LiveView 核心的技术机制

1. 页面初始加载是 HTML 渲染(传统方式)
2. 后续交互走 WebSocket，每个事件都是消息 (phx-* 事件) 传回后端服务器处理
3. LiveView 内部维护 socket 状态，每次更新后只发送最小化 diff patches
4. 客户端接收到数据后，通过 LiveView 内置的 JS 框架对 DOM 结构进行更新，调用相应的 JS hooks (phx-hook)

![](/images/posts/26_phoenix_liveview_intro/liveview_2.png)

# 引用

- [LiveView HexDoc](https://hexdocs.pm/phoenix_live_view/welcome.html)
- [Phoenix LiveView 1.0.0 is here!](https://www.phoenixframework.org/blog/phoenix-liveview-1.0-released)
- [Phoenix LiveView 简介](https://yiming.dev/blog/2020/05/17/an-introduction-to-phoenix-live-view/)
- [phoenix_live_view_example](https://github.com/chrismccord/phoenix_live_view_example)