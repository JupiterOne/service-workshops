Since InspectorV2's release, I've been asked about an interesting limitation in the AWS InspectorV2 API. The service has alot of different relationships and vulnerability types, but the most common of interest I've encountered are the package vulnerabilities that relate to the EC2 service.

Inspector in J1 gives you a couple ways to identify those using:
```FIND aws_inspectorv2_finding with type="PACKAGE_VULNERABILITY" and resourceType="AWS_EC2_INSTANCE"```

One thing to note, I could have also typed:
```FIND aws_inspectorv2_finding with type="PACKAGE_VULNERABILITY" THAT HAS aws_instance```

The query above results in similar results as the first one.

When someone starts digging into the PACKAGE_VULNERABILITY findings from inspectorv2, a conclusion usually is drawn upon. "Why am I focused on the instances that have these findings, when I should be looking at the AMIs?!" 

Why would this be the most optimal conclusion? For many organizations, the AMI is going to be the source of the problems found by inspectorV2 findings. Since a single AMI might be used across 100 or more EC2 instances, it stands to reason that if you're seeing the same Iv2 Finding across those 100 instance, the AMI is the real source of the finding.
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instances-and-amis.html

In J1, we are relating these findings to the EC2 Instance. If we wanted to get a view into how many of these findings are relating directly to a single AMI, we could use the query below:

```FIND aws_instance  AS a that has aws_inspectorv2_finding as f return a.imageId, count(f) as Total ORDER BY Total DESC```

We could also create some variations on that query that might help us identify the most important finding/AMI pair across our J1 Data.

#### Critical/Production Impacted AMI/Findings 
```FIND aws_instance with tag.Production = true  AS a that has aws_inspectorv2_finding  with severity = 'critical' as f return a.imageId, count(f) as Total ORDER BY Total DESC```

We could also look at CVEs most prevlant across our AMI using the following query:

#### Most Common CVE across AMIs
```FIND aws_instance  AS a that has aws_inspectorv2_finding as f  return f.displayName, count(a.imageId) as Total ORDER BY Total DESC```
