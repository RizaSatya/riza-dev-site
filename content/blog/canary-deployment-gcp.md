---
title: "[DRAFT] Building a Canary Deployment Infrastructure in GCP"
# description: "How I passed the AWS Developer - Associate Certification Exam (DVA-C02)"
dateString: July 2024
draft: false
tags: ["GCP", "Terraform"]
weight: 101
cover:
    image: "/blog/canary-deployment-gcp/cover.png"
---

# Introduction
Let's face it, pushing new code to production can be nerve-wracking. One tiny bug and suddenly you're putting out fires instead of sipping your well-earned coffee. That's where canary deployments come in handy. They let you dip your toes in the water before diving in headfirst.
In this article, we're going to walk through setting up a canary deployment system on Google Cloud Platform (GCP) using Terraform. We'll be working with regional managed instance groups, load balancers, and some traffic-splitting magic to get a solid canary setup going.

While continuous deployment tools like Chef or Ansible are often part of a complete canary strategy, they're outside the scope of what will be discussed here. The goal is to share insights on creating a robust infrastructure base that can support canary deployments, which can then be integrated with various deployment tools and processes.

# What is a Canary Deployment
Canary deployment is a way to safely roll out new software versions. Instead of updating everything at once, you release the new version to a small group of users first. This lets you test the waters and catch any problems early. If everything looks good, you gradually increase the number of users getting the new version.

Let's take a look at the following analogy, imagine you're a chef introducing a new recipe at your restaurant. Instead of immediately replacing your popular dish with this new recipe for all customers, you decide to take a cautious approach:
1. You prepare a small batch of the new recipe alongside your usual menu.
2. When customers order, most get the original dish, but a small percentage (let's say 10%) receive the new recipe.
3. You closely watch for feedback. Are the customers who got the new dish enjoying it? Are there any complaints or allergic reactions?
4. If the new recipe is well-received, you gradually increase the number of customers who get it, maybe to 25%, then 50%, and so on.
5. If there are issues with the new recipe, you can quickly stop serving it and revert to the original dish for all customers, minimizing the impact.
6. Once you're confident the new recipe is a hit, you fully replace the old dish with the new one.

This approach is essentially what a canary deployment does in software:
- The original dish is your stable version of the software.
- The new recipe is your updated version.
- The gradual rollout to a small percentage of customers is how you introduce the new version to a subset of users.
- Monitoring feedback is like watching for errors or performance issues in your application.
- The ability to quickly revert is your safety net if something goes wrong.

Just as this approach lets a chef test a new recipe without risking the satisfaction of all customers, a canary deployment allows developers to test new software versions with minimal risk to the overall user base.

# Architecture Diagram
![](/blog/canary-deployment-gcp/img1.png)

# Components
- HTTP Load Balancer


- Forwarding Rule
```terraform
resource "google_compute_forwarding_rule" "default" {
  name                  = "http-lb-forwarding-rule"
  region                = "us-central1"
  target                = google_compute_region_target_http_proxy.default.id
  port_range            = "80"
  load_balancing_scheme = "INTERNAL_MANAGED"
  network               = "default"
  subnetwork            = google_compute_subnetwork.proxy_only.id
}
```

- HTTP Target Proxy
```terraform
resource "google_compute_region_target_http_proxy" "default" {
  name    = "http-lb-proxy"
  region  = "us-central1"
  url_map = google_compute_region_url_map.default.id
}
```
- URL Map

This is where we implement the traffic split. We use weighted_backend_services to control the distribution of requests. By setting the weight for stable to 80 and canary to 20, we ensure that 80% of the incoming traffic is directed to the stable version, while 20% goes to the canary version. This allows us to gradually expose users to the new version while maintaining the majority of traffic on the proven, stable version.

```terraform
resource "google_compute_region_url_map" "default" {
  name            = "url-map"
  default_service = google_compute_region_backend_service.stable.id
  region          = "us-central1"

  host_rule {
    hosts        = ["*"]
    path_matcher = "allpaths"
  }

  path_matcher {
    name            = "allpaths"
    default_service = google_compute_region_backend_service.stable.id

    route_rules {
      priority = 1
      match_rules {
        prefix_match = "/"
      }
      route_action {
        weighted_backend_services {
          backend_service = google_compute_region_backend_service.stable.id
          weight          = 80
        }
        weighted_backend_services {
          backend_service = google_compute_region_backend_service.canary.id
          weight          = 20
        }
      }
    }
  }
}
```

- Canary Instance Template
```terraform
resource "google_compute_instance_template" "canary" {
  name         = "canary-template"
  machine_type = "e2-micro"

  disk {
    source_image = "debian-cloud/debian-11"
    auto_delete  = true
    boot         = true
  }

  network_interface {
    network = "default"
    access_config {}
  }

  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    echo "Canary Version" > /var/www/html/index.html
  EOF

  tags = ["http-server"]
}
```

- Canary Managed Instance Group
```terraform
resource "google_compute_region_instance_group_manager" "canary" {
  name               = "canary-mig"
  base_instance_name = "canary"
  region             = "us-central1"
  target_size        = 1

  version {
    instance_template = google_compute_instance_template.canary.id
  }

  named_port {
    name = "http"
    port = 80
  }
}
```

- Stable Instance Template
```terraform
resource "google_compute_instance_template" "stable" {
  name         = "stable-template"
  machine_type = "e2-micro"

  disk {
    source_image = "debian-cloud/debian-11"
    auto_delete  = true
    boot         = true
  }

  network_interface {
    network = "default"
    access_config {}
  }

  metadata_startup_script = <<-EOF
    #!/bin/bash
    apt-get update
    apt-get install -y nginx
    echo "Stable Version" > /var/www/html/index.html
  EOF

  tags = ["http-server"]
}
```

- Stable Managed Instance Group
```terraform
resource "google_compute_region_instance_group_manager" "stable" {
  name               = "stable-mig"
  base_instance_name = "stable"
  region             = "us-central1"
  target_size        = 2

  version {
    instance_template = google_compute_instance_template.stable.id
  }

  named_port {
    name = "http"
    port = 80
  }
}
```



DRAFT
DRAFT
DRAFT
DRAFT
DRAFT
DRAFT