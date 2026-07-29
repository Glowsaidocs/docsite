---
id: faqs
sidebar_position: 19
---

# Glows.ai GPU Rental Platform FAQs

This document answers common questions about the Glows.ai GPU rental platform, including data storage, instance environments, connection methods, automated deployment, billing, and service guarantees. If you need further assistance, please contact us through the official support channels.

## DataDrive and Snapshot Data Management

### 1. Can I release an instance to stop compute charges and only keep storage charges?

Yes. When you do not need to run training or inference workloads, you can release the instance to stop GPU compute billing and use DataDrive or Snapshot to preserve the data or environment you need.

- Models, datasets, code, and other data files are recommended to be stored in DataDrive. DataDrive supports upload and download without starting an instance.
- Python packages, system software, environment settings, and other reusable environment changes should be saved by clicking `Take Snapshot` before releasing the instance.
- A Snapshot saves changes inside the instance, excluding DataDrive. Smaller Snapshots usually make it faster to create new instances later.

DataDrive and Snapshot require a storage plan on the `Storage Space` page. You can allocate capacity between DataDrive and Snapshot based on your actual needs.

### 2. Where can I view files uploaded to DataDrive?

Select DataDrive mounting when creating an instance. After logging in to the instance, files are available under `/datadrive`.

```bash
ls /datadrive
```

### 3. Can multiple instances share the same DataDrive?

Yes. Instances in the same region can share data by mounting the same DataDrive. We recommend creating a test instance first to confirm that the image environment, paths, and permissions fit your workflow.

### 4. Will data on the instance disk be deleted after the instance is released or the rental period ends?

Yes. The instance disk is only valid while the instance is running. Data on the instance disk is removed after the instance is released.

We recommend storing important data in DataDrive. If you need to keep the environment or installed software, create a Snapshot before releasing the instance. If you do not use platform storage, you can also download files to your local machine with tools such as `scp` before releasing the instance.

### 5. Will DataDrive and Snapshot be automatically deleted after the storage plan expires?

They will not be deleted immediately. DataDrive and Snapshot are managed through storage plans. After a plan expires, data may remain in a recoverable state, but you need to renew the plan to continue normal usage. To avoid service impact, we recommend renewing before the plan expires.

### 6. After rebooting an instance with DataDrive mounted, `/datadrive` disappeared. Is my data lost?

No. DataDrive data is not lost because of an instance reboot. If DataDrive is not mounted automatically after reboot, you can try mounting it manually inside the instance:

```bash
sudo mount -t virtiofs DataDrive /datadrive
```

This usually happens when auto-mount is not triggered during reboot. We will continue improving this workflow.

### 7. How can I use Snapshot more efficiently?

Snapshot saves changes on the instance disk. We recommend keeping environments, packages, and system settings on the instance disk, while storing datasets, model weights, and code in DataDrive. This helps reduce Snapshot size and improves the speed of creating new instances from Snapshot.

When using Snapshots across regions, instance creation speed can be affected by Snapshot size and network transfer conditions. Keeping Snapshots lightweight is recommended.

### 8. Can the DataDrive web interface upload an entire folder at once?

The DataDrive web interface currently mainly supports single-file uploads. To upload a folder, compress it into a single archive file first or use the DataDrive PC version.

DataDrive PC version download: [https://glows.ai/datadrive](https://glows.ai/datadrive)

### 9. What should I do if uploading a large file through the DataDrive web interface fails?

We recommend using the DataDrive PC version. It supports Windows and macOS and is suitable for uploading or downloading large files and folders.

Download: [https://glows.ai/datadrive](https://glows.ai/datadrive)

### 10. Can DataDrive data be synced directly across regions?

Cross-region DataDrive sync is not currently supported. For example, to move data from TW-01 to TW-02, download it locally first and then upload it to the DataDrive in the target region.

### 11. Can Snapshots be exported?

No. Snapshot is an internal format used by the Glows.ai platform and does not currently support direct export.

### 12. Can I upload or download data without starting an instance?

Yes. DataDrive supports offline upload and download without starting a GPU instance.

- DataDrive tutorial: [Datadrive](https://docs.glows.ai/docs/datadrive)
- DataDrive PC version: [Download](https://glows.ai/datadrive)

### 13. Can I add or mount a DataDrive to a running VM?

Not currently. Please select the DataDrive you need when creating the instance.

### 14. I cleared DataDrive, but the Storage page still shows used capacity. Why?

After deleting data through the DataDrive interface, the Storage page usually needs 5-10 minutes to sync the status. To confirm actual usage, create an instance in the same region, mount DataDrive, and run:

```bash
# View total storage usage
df -h | grep datadrive

# View files and folders under /datadrive
ls -alh /datadrive
```

Ubuntu may temporarily place some deleted files under `/datadrive/.Trash-0`. If you are sure the files are no longer needed, run the following command to delete them permanently:

```bash
# This action cannot be undone. Confirm the path before running it.
rm -rf /datadrive/.Trash-0
```

### 15. What does `Disk quota exceeded` mean when writing to DataDrive?

This error usually means there is not enough storage capacity. Common causes include:

1. DataDrive capacity is full. Clean up unnecessary files or expand the capacity on the Storage page.
2. The Storage plan has expired. Renew the plan to continue using it.

### 16. How can I downgrade a Storage plan during renewal?

To downgrade from a higher-capacity plan to a lower-capacity plan, make sure your actual usage is below the target plan capacity first.

For example, if your current plan is 300GB and you want to renew with a 200GB plan, reduce actual usage to below 200GB by deleting unnecessary files from DataDrive, Snapshot, or Images.

![Storage usage before cleanup](../docs-images/p20faqs/01.png)

After cleanup, confirm that usage is below the target plan capacity.

![Storage usage after cleanup](../docs-images/p20faqs/02.png)

Click `Modify`, adjust the allocated total capacity to be less than or equal to the target plan capacity, and then click `Update`.

![Modify storage allocation](../docs-images/p20faqs/03.png)

After that, click `Upgrade` and select the lower-capacity plan.

![Select lower storage plan](../docs-images/p20faqs/04.png)

### 17. What does the `.partial` file extension mean?

`.partial` is a temporary file generated by the system during upload. It disappears automatically after the upload is complete and does not affect normal usage.

## Image and Storage Space Management

### 1. How do I upload a custom Docker image?

See the official guide: [Upload Custom Docker Image](https://docs.glows.ai/docs/upload-custom-docker-image)

### 2. What should I do if an image upload is interrupted because the file is large, the network is unstable, or the page is switched?

The platform supports resumable uploads. Even if the upload is interrupted, it can continue from where it stopped without starting over.

### 3. How can I transfer local models and Python code into a Glows.ai instance?

We recommend using DataDrive first:

1. Upload local files to DataDrive.
2. Mount DataDrive when creating the instance.
3. Log in to the instance and read the files under `/datadrive`.

If the instance is already running, you can also use `scp`. Example for transferring from a Windows local machine to `/home` on the instance:

```bash
scp -P 23675 -r C:\Users\Data root@tw-03.access.glows.ai:/home
```

Each instance may have a different SSH address and port. Use the information shown on the instance page.

### 4. Does the Glows.ai Snapshot feature preserve installed programs and data?

Yes. Snapshot saves the files, settings, installed software, and configuration inside the instance at the time the Snapshot is created, excluding data in DataDrive. DataDrive data is already stored in the cloud and does not need to be saved into Snapshot again.

## Image and Application Usage

### 1. I generated a video with tools such as FramePack but cannot download it. What should I do?

First confirm that the instance has not been released. Then try downloading the file manually through JupyterLab:

1. Open JupyterLab for the instance.
2. Go to `/FramePack/outputs/`.
3. Find the generated `.mp4` file.
4. Right-click the file and select `Download`.

If the issue persists, take a screenshot and contact platform technical support.

### 2. ComfyUI cannot use an imported workflow because custom nodes are missing. What should I do?

Create an instance, load the workflow, and provide the error screen or missing-node message. Platform technical support can help identify the custom nodes or dependencies that need to be installed.

### 3. ComfyUI shows a missing Model warning when testing WAN 2.1. What does it mean?

It usually means the model has not been loaded correctly. Check the model file path, workflow settings, and model name. If the issue remains, provide a screenshot for technical support.

## Instance Environment, Permissions, and Connectivity

### 1. Is Docker or K8s supported? What is the difference between VM and Container?

Glows.ai provides two underlying cloud service types: VM and Container.

- Container instances provide more preconfigured base environments and are easier to set up, but they do not support running Docker inside the instance or modifying low-level software such as NVIDIA drivers.
- VM instances provide higher privileges and support custom GPU drivers, Docker, Systemctl, and other system-level capabilities. They are suitable for workloads that require full system control.

When creating an instance, selecting `Windows` or a `For VM` image creates a VM instance.

![VM image selection](../docs-images/p20faqs/05.png)

### 2. Do instances have root access?

Yes. Instances log in as the root account by default. You can also create additional users after logging in.

### 3. Can I choose the OS environment?

The platform currently provides Ubuntu-series OS environments and Windows Server 2025. Ubuntu is recommended for most use cases. If you need another OS, contact us for requirement evaluation.

### 4. Can Windows remote operation be laggy or unstable?

Windows remote desktop performance can be affected by network conditions, the graphical interface, and workload. If your task does not depend on Windows, Linux is recommended for a more stable training and inference experience.

### 5. What remote connection methods are supported?

Ubuntu instances support SSH and JupyterLab. Windows instances support RDP. We recommend using the connection methods provided by the platform first. Other connection tools can be installed after logging in if needed.

### 6. Does `tw-02.access.glows.ai` in the RDP connection URL indicate the actual instance region?

Not necessarily. `tw-02` in the connection URL may be an access node code and does not represent the actual compute host region. Please refer to the deployment region shown on the instance page.

### 7. Can I monitor GPU/CPU usage, memory, and disk usage in real time?

Yes. After an instance is created, the instance page provides a Monitor feature showing CPU/GPU utilization, GPU memory, system memory, disk usage, and related metrics over time.

![Instance monitor](../docs-images/p20faqs/06.png)

### 8. Can customers update NVIDIA Driver themselves?

It depends on the instance type:

1. Container instances do not currently support manual NVIDIA Driver replacement.
2. VM instances allow customers to download and update Driver themselves. The platform can provide technical support when needed. [Contact Us](https://docs.glows.ai/docs/contact-us)

![NVIDIA driver example](../docs-images/p20faqs/07.png)

## Autodeploy, Networking, and Enterprise Features

### 1. Is auto-scaling supported?

Yes. The platform provides Autodeploy, which can automatically create an instance when a request arrives, start the service, and release the instance after it becomes idle.

Typical workflow:

1. A customer sends a request to the Autodeploy link.
2. The Glows.ai backend creates an instance based on your configuration.
3. The configured Service Start Command runs.
4. The request is forwarded to your configured service port.
5. The result is returned.
6. If no new request is received for n minutes, the platform automatically releases the instance.

You can also use the Glows.ai SDK to programmatically control instance startup and release.
Autodeploy guide: [Autodeploy](https://docs.glows.ai/docs/auto-deploy-usage)
SDK guide: [SDK Docs](https://sdkdoc.glows.ai)

### 2. Is CLI or API-based automated deployment available?

Enterprise customers with API-based automated deployment needs can contact us for evaluation. SDK guide: [SDK Docs](https://sdkdoc.glows.ai)

### 3. Can I use APIs to start or stop machines to avoid idle costs?

API functionality is currently mainly available for enterprise customers with automation needs. Please contact us through the official support channels to discuss access.

### 4. Does the platform charge for network traffic? Are fixed bandwidth or static IP options available?

In general, the platform does not separately charge for data transfer. If you require dedicated 1Gbps bandwidth, static IP, or other advanced networking capabilities, you can apply for an add-on plan. Sales and technical teams will confirm the requirements before activation.

### 5. Will IP/DNS change after an instance is shut down or restarted?

It may change. If you need a more stable service endpoint, we recommend using Autodeploy fixed service links and designing your own traffic distribution strategy.

### 6. Can enterprise private network segments or VPN-restricted access be used?

This capability is under development and planning. In the future, enterprise customers are expected to be able to configure allowed network IPs, such as limiting SSH access to a specific VPN range. Official availability depends on platform announcements.

### 7. Can existing servers or GPU machines be integrated into Glows.ai?

Hybrid cloud or private cloud deployment can be evaluated to integrate local CPU/GPU machines as part of the compute resource pool. Feasibility depends on hardware, network, and deployment requirements and must be confirmed by the technical team.

Contact us if interested: [Contact Us](https://docs.glows.ai/docs/contact-us)

### 8. Can private cloud include CPU machines for unified management?

Yes. Private cloud solutions can integrate local non-GPU machines and provide unified management and scheduling. The exact solution requires project evaluation.

## GPU Resources and Infrastructure Support

### 1. Do H100/H200 GPUs support NVIDIA MIG partitioning?

H100/H200 GPUs deployed on Glows.ai support NVIDIA MIG capabilities. Actual availability and configuration depend on the product plan and resource conditions.

### 2. Are GPUs rented as whole GPUs?

The platform currently mainly provides whole-GPU rental.

### 3. Is NCCL supported between GPUs?

Yes. NCCL is supported for distributed training and multi-GPU data synchronization.

## Account, Billing, and Top-up

### 1. Can I register with a regular email instead of Gmail?

The system currently supports Gmail registration by default. If you need to create an account with a regular email address, please contact the technical or sales team for evaluation and assistance.

### 2. Is there a minimum online top-up amount?

Yes. The minimum online top-up amount is USD 10, in increments of USD 10.

### 3. What happens if I forget to top up and my credits run out?

When credits are depleted, the system automatically creates a Snapshot for running instances and shuts them down. After topping up again, you can restore the environment from the automatically generated Snapshot.

### 4. Is there a minimum rental duration?

No. The platform uses pay-as-you-go billing. You pay for what you use. When instances are not running, GPU compute billing stops and only storage fees apply based on actual usage.

### 5. How much does an H200 GPU cost?

Please refer to the platform page for the current pay-as-you-go H200 GPU rate. For long-term or monthly rental, contact us to discuss discounted plans.

### 6. How is storage priced?

Storage is billed by plan or usage. Please refer to the Storage page for current pricing. For bulk purchase or enterprise requirements, contact us to discuss pricing.

### 7. Are bulk or long-term rental discounts available?

Yes. For bulk or long-term rental, please contact us: [Contact Us](https://docs.glows.ai/docs/contact-us)

### 8. Is there a subscription fee, setup fee, or minimum usage fee?

No. For general usage, there is no subscription fee, setup fee, or minimum usage fee.

### 9. How are software or platform partners billed?

Pricing can be evaluated based on usage volume, storage volume, and partnership model. For agency, resale, or platform cooperation, please contact the sales team to discuss an agreement.
