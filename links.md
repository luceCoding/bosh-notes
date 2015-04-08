# Links

To configure non-simplistic deployment where at least one deployment job knows about another, operators have to either assign static IPs or DNS names to one job and pass it via properties to the other. This configuration is error-prone and unnecessary. It is also hard to automate for the case of service brokers creating deployments on demand.

Currently the Director favors deployments/environments with manual networking (the Director assigns IPs to the deployment jobs) because it does not come with a HA DNS service. (See separate proposal regarding HA DNS.) In addition to that limitation even if HA DNS was provided operators would still have to configure certain deployment jobs to have static IPs.

Links provide a solution to the above problem and abstract away manual vs dynamic (DNS based networking) from the deployment jobs. In addition to that links can also be used to share other non-networking configuration (job properties) between deployment jobs.

From the operator perspective introduction of links removes tedious cross-referencing of IPs and in future other properties.

## Changes to Releases

To take advantage of links in releases, extra metadata needs to be specified: `requires` and `provides` directives. Each deployment job that needs information about another job has to specify `requires` with the name of the link type it requires. Each deployment job that can satisfy link type has to specify `provides`. For example, in the MySQL release:

MySQL proxy job `spec`:

```yaml
name: proxy
requires: [data-node]
```

MySQL node job `spec`:

```yaml
name: node
requires: [data-node]
provides: [data-node]
```

Proxy job requires information about all nodes and each node job provides it. Also each node needs to know about other nodes, hence, node job requires and provide node.

To access information provided by the link, release author would modify their ERB template. For example MySQL proxy job may do the following:

```yaml
nodes: <%= p("data-node.nodes") %> # todo pick out IPs in a network?

# Link provided properties can be access like before: p, if_p, etc.
admin_user: <%= p("data-node.admin_user") %>
public_key: <%= p("data-node.public_key") %>
```

Link information contains following:

```yaml
{
	"nodes": [
		# For each one of the deployment job instances
		{
			"name": "data-node",
			"index": 0,
			"availability_zone": "z1",
			"networks": {
				"private": { "address": "10.0.0.44" || "0.private.data-node.deployment" || "IPv6" },
				"external": { "address": "56.34.78.101" || "0.external.data-node.deployment" || "IPv6" }
			}
		},
		{
			"name": "data-node",
			"index": 1,
			"availability_zone": "z1",
			"networks": {
				"private": { "address": "10.0.0.45" || "1.private.data-node.deployment" || "IPv6" },
				"external": { "address": "101.34.78.10" || "1.external.data-node.deployment" || "IPv6" }
			}
		}
	],

	# Link provided properties
	"admin_user": "admin-user",
	"admin_password": "some-secret",
	"public_key": "..."
}
```

## Changes to Deployment Manifests

### Link Resolution

- multiple links types provided by different releases
- multiple links of the same type (primary_db and backup_db links)
- links between deployments
- custom provide links
- links that only require access to a single collocated deployment job

## Stories

## TBD