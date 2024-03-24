---
layout: post
title: Michigan Tech Conf
date: 2024-03-24 17:10:00 -0500
categories: Michigan Technology
---

### Reflections from the First Annual Michigan Technology Conference

This past week, I attended the first [Michigan Tech Conference](https://www.mitechcon.org/), a gathering that brought in some of the most brilliant tech minds in from across Michigan and beyond. It was an inspiring confluence of innovation and networking. Below are my takeaways and insights.

#### The Keynote: The Myopia of Platform Companies

The keynote address, delivered by Tim O'Relly, critiqued that platform companies' short-sighted focus on extracting immediate value at the expense of long-term growth. This strategy, he argued, erodes the relationships with suppliers and users, paving the way for competitors. Bill Gates's perspective, as shared by [Stratechery](https://stratechery.com/2018/the-bill-gates-line/), that  "A platform is when the economic value of everybody that uses it, exceeds the value of the company that creates it. Then it’s a platform," resonates here. As generative AI evolves, it will be interesting to observe if companies like OpenAI can indeed prioritize their customers' value over their own.

![Tim O'Relly at Michigan Technology Conference]({{ BASE_PATH }}/images/michigan-tech-conf/IMG_5446.jpeg)

#### A Brush with Async

Meeting Stephen Cleary in person was a highlight. His insights into asynchronous operations in .NET, detailed in his [blog](https://blog.stephencleary.com), are incredibly valuable. Stephen's deep understanding emphasizes the power of specialization in technology, especially with the emergence of AI challenging developers to stay relevant.

#### OpenAI, Azure, and Beyond

The session on the latest [Azure AI Studio](https://azure.microsoft.com/en-us/products/ai-studio) showcased the potential of combining OpenAI's capabilities with Microsoft's platform, simplifying AI application development to low-code level. Yet, it's an open question how much additional value Azure AI Studio will offer to entice users from competing cloud platforms.

*Azure AI Studio is currently under public review.*

#### GitHub Copilot's Two-Part Measures Against Regurgitation

GitHub's proactive measures to prevent Copilot from regurgitating training data are noteworthy. Through screening user requests via a separate large language model, GitHub filters out requests intended to coax the model to reveal its training information. In addition, GitHub conducts further reviews to ascertain if its outputs inadvertently include elements of the training data. Both actions can be executed instantaneously. It prompts curiosity about whether OpenAI is adopting similar measures, especially considering lawsuit with the New York Times.

#### Google's BeyondCorp and Operation Aurora

The presentations on Google's [BeyondCorp](https://cloud.google.com/beyondcorp) were informational. Google moved away from the traditional VPN approach to Zero Trust Architecture. The fundamental flaw with VPNs is their inherent trust in all requests originating from the internal network. Conversely, Zero Trust Model scrutinizes every request, irrespective of its origin, based on the user's identity, the device used, permissions, and other factors. This strategic pivot was Google's response to [Operation Aurora](https://en.wikipedia.org/wiki/Operation_Aurora), a cyber attack launched by a national government in 2010. Personally, I'm grateful that the presenter did not explicitly name the nation involved.

<iframe width="560" height="315" src="https://www.youtube.com/embed/TtmsV-xq0r0?si=dMr-iST4kINnJMus" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

#### Terraform Remote State Management

Discussing remote state management with a Hashicorp engineer led me to reconsider how we handle Terraform states. Commonly, cloud-based blob storage solutions like AWS S3 are used for storing Terraform's state files remotely. This approach is not only more secure than storing on a local device but also facilitates collaborative deployment modifications by preventing conflicts. For AWS users, ensuring the security of the S3 bucket is thus crucial, with recommended practices including:

- Enable versioning to maintain a historical record of state files, which is invaluable for recovering previous versions if necessary due to loss or damage.
- Use Private Endpoint to permit access to the S3 buckets only from the VPC, avoiding the public internet completely.
- Use S3 access logs to monitor and identify any unauthorized attempts to access or update the state files.

Alternatively, Terraform Cloud offers a comprehensive solution for remote state management, eliminating the need for additional configurations like a separate DynamoDB table for state locking, which is a requirement when using AWS S3. Terraform Cloud serves as an all-in-one platform, streamlining the process.


#### Conclusion: The Value of Being There

Attending the Michigan Tech Conference was an enriching experience that broadened my horizons. While the internet offers a vast reservoir of information, knowing what to search for is often the biggest hurdle. This conference not only provided a curated platform for learning and discovery but also emphasized the irreplaceable value of in-person interactions and the exchange of ideas. It's a reminder that in the rapidly evolving tech landscape, staying connected and informed through such forums is invaluable.