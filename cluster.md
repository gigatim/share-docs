---
layout: doc
toc: true
title: Clustering
description: How Gigalixir Manages Clustering Nodes with Libcluster and How to Set up Distributed Phoenix Channels after successfully clustering your nodes | Gigalixir
order: 150
permalink: /docs/cluster
aliases: 
  - /docs/cluster.html
---
## Clustering Nodes

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; height: auto; margin-bottom: 20px;">
    <iframe src="https://player.vimeo.com/video/536159871" frameborder="0" allowfullscreen style="position: absolute; top: 0; left: 0; width: 100%; height: 100%;"></iframe>
</div>

We use libcluster to manage node clustering. For more information, see
[libcluster's documentation](https://github.com/bitwalker/libcluster){:target="\_blank"}.

To install libcluster, add this to the deps list in `mix.exs`:

``` elixir
{:libcluster, "~> 3.2"}
```

Next, add the following to the existing `start` function in your `application.ex` file. 

Remember to replace `GigalixirGettingStarted` with your application name.

``` elixir
def start(_type, _args) do
  topologies = Application.get_env(:libcluster, :topologies) || []

  children = [
    {Cluster.Supervisor, [topologies, [name: GigalixirGettingStarted.ClusterSupervisor]]},
    ... # other children
  ]
  ...
end
```

Your app configuration needs to have something like this in it. For a
full example, see [gigalixir-getting-started's prod.exs file](https://github.com/gigalixir/gigalixir-getting-started/blob/ff56b063b4bb2519acd3dc82893ce6accd714d8e/config/prod.exs#L33){:target="\_blank"}.

``` elixir
...
config :libcluster,
  topologies: [
    k8s_example: [
      strategy: Cluster.Strategy.Kubernetes,
      config: [
        kubernetes_selector: System.get_env("LIBCLUSTER_KUBERNETES_SELECTOR"),
        kubernetes_node_basename: System.get_env("LIBCLUSTER_KUBERNETES_NODE_BASENAME")]]]
...
```

Gigalixir handles permissions so that you have access to Kubernetes endpoints and we automatically set your node name and Erlang cookie so that your nodes can reach each other. 

We don't firewall each container from each other like Heroku does. We also automatically set the environment variables `LIBCLUSTER_KUBERNETES_SELECTOR`, `LIBCLUSTER_KUBERNETES_NODE_BASENAME`, `APP_NAME`, and `MY_POD_IP` for you. 

## How to Set Up Distributed Phoenix Channels

If you have successfully clustered your nodes, then distributed Phoenix channels *just work* out of the box. 

No need to follow any of the steps described in [Running Elixir and Phoenix projects on a cluster of nodes](https://dockyard.com/blog/2016/01/28/running-elixir-and-phoenix-projects-on-a-cluster-of-nodes){:target="\_blank"}.

See more information on how to [cluster your nodes](/docs/cluster#clustering-nodes){:target="\_blank"}.
