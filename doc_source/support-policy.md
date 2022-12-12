# Framework Support Policy<a name="support-policy"></a>

[AWS Deep Learning Containers](http://aws.amazon.com/machine-learning/containers/?nc2=h_ql_prod_ml_con) \(DLCs\) simplify image configuration for deep learning workloads and are optimized with the latest frameworks, hardware, drivers, libraries, and operating systems\. This page details the framework support policy for DLCs\. For a list of available DLCs, see [Release Notes for Deep Learning Containers](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/dlc-release-notes.html)\. 

## Supported Frameworks<a name="supported-frameworks"></a>

Reference the following [AWS Deep Learning Containers Framework Support Policy table](https://aws.amazon.com/releasenotes/dlc-support-policy/) to **check which frameworks and versions are actively supported**\. 

Refer to **End of patch** to check how long AWS supports current versions that are actively supported by the origin framework’s maintenance team\. Frameworks and versions are available in single\-framework DLCs\. 

**Note**  
In the framework version *x\.y\.z*, *x* refers to the major version, *y* refers to the minor version, and *z* refers to the patch version\. For example, for TensorFlow 2\.6\.5, the major version is 2, the minor version is 6, and the patch version is 5\.

Refer to the release notes for more details on specific images:
+ [Single\-framework DLC](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/dlc-release-notes.html#appendix-dlc-release-notes-frameworks) release notes
+ [Available Deep Learning Containers Images](https://github.com/aws/deep-learning-containers/blob/master/available_images.md) page

## Frequently Asked Questions<a name="support-policy-faq"></a>
+ [What framework versions get security patches?](#support-policy-faq-security)
+ [What images does AWS publish when new framework versions are released?](#support-policy-faq-publishing)
+ [What images get new SageMaker/AWS features?](#support-policy-faq-features)
+ [How is current version defined in the Supported Frameworks table?](#support-policy-faq-current-version)
+ [What if I am running a version that is not in the Supported Frameworks table?](#support-policy-faq-older-version)
+ [Do DLCs support previous versions of TensorFlow?](#support-policy-faq-previous-version-support)
+ [How can I find the latest patched image for a supported framework version?](#support-policy-faq-latest-patched-image)
+ [How frequently are new images released?](#support-policy-faq-new-image-frequency)
+ [Will my instance be patched in place while my workload is running?](#support-policy-faq-in-place-patch)
+ [What happens when a new patched or updated framework version is available?](#support-policy-faq-new-image-available)
+ [Are dependencies updated without changing the framework version?](#support-policy-faq-dependencies)
+ [When does active support for my framework version end?](#support-policy-faq-end-of-support)
+ [Will images with framework versions that are no longer actively maintained be patched?](#support-policy-faq-end-of-patch)
+ [How do I use an older framework version?](#support-policy-faq-using-older-framework-version)
+ [How do I stay up\-to\-date with support changes in frameworks and their versions?](#support-policy-faq-stay-up-to-date)
+ [Do I need a commercial license to use the Anaconda Repository?](#support-policy-faq-anaconda-repository)

### What framework versions get security patches?<a name="support-policy-faq-security"></a>

If the framework version is labeled **Supported** in the [AWS Deep Learning Containers Framework Support Policy table](https://aws.amazon.com/releasenotes/dlc-support-policy/), it gets security patches\. 

### What images does AWS publish when new framework versions are released?<a name="support-policy-faq-publishing"></a>

We publish new DLCs soon after new versions of TensorFlow and PyTorch are released\. This includes major versions, major\-minor versions, and major\-minor\-patch versions of frameworks\. We also update images when new versions of drivers and libraries become available\. For more information on image maintenance, see [When does active support for my framework version end?](#support-policy-faq-end-of-support)

### What images get new SageMaker/AWS features?<a name="support-policy-faq-features"></a>

New features typically release in the latest version of DLCs for PyTorch and TensorFlow\. Refer to the release notes for a specific image for details on new SageMaker or AWS features\. For a list of available DLCs, see [Release Notes for AWS Deep Learning Containers](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/dlc-release-notes.html)\. For more information on image maintenance, see [When does active support for my framework version end?](#support-policy-faq-end-of-support)

### How is current version defined in the Supported Frameworks table?<a name="support-policy-faq-current-version"></a>

The current version in the [AWS Deep Learning Containers Framework Support Policy table](https://aws.amazon.com/releasenotes/dlc-support-policy/) refers to the newest framework version that AWS makes available on GitHub\. Each latest release includes updates to the drivers, libraries, and relevant packages in the DLC\. For information on image maintenance, see [When does active support for my framework version end?](#support-policy-faq-end-of-support)

### What if I am running a version that is not in the Supported Frameworks table?<a name="support-policy-faq-older-version"></a>

If you are running a version that is not in the [AWS Deep Learning Containers Framework Support Policy table](https://aws.amazon.com/releasenotes/dlc-support-policy/), you may not have the most updated drivers, libraries, and relevant packages\. For a more up\-to\-date version, we recommend that you upgrade to one of the supported frameworks available using the latest DLC of your choice\. For a list of available DLCs, see [Release Notes for AWS Deep Learning Containers](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/dlc-release-notes.html)\.

### Do DLCs support previous versions of TensorFlow?<a name="support-policy-faq-previous-version-support"></a>

No\. We support the latest patch version of each framework’s latest major version released 365 days from its initial GitHub release as stated in the [AWS Deep Learning Containers Framework Support Policy table](https://aws.amazon.com/releasenotes/dlc-support-policy/)\. For more information, see [What if I am running a version that is not in the Supported Frameworks table?](#support-policy-faq-older-version)

### How can I find the latest patched image for a supported framework version?<a name="support-policy-faq-latest-patched-image"></a>

To use a DLC with the latest framework version, browse the [DLC GitHub release tags ](https://github.com/aws/deep-learning-containers/releases)to find the sample image URI of your choice and use it to pull the latest available Docker image\. The framework version that you choose must be labeled **Supported** in the [AWS Deep Learning Containers Framework Support Policy table](https://aws.amazon.com/releasenotes/dlc-support-policy/)\.

### How frequently are new images released?<a name="support-policy-faq-new-image-frequency"></a>

Providing updated patch versions is our highest priority\. We routinely create patched images at the earliest opportunity\. We monitor for newly patched framework versions \(ex\. TensorFlow 2\.9 to TensorFlow 2\.9\.1\) and new minor release versions \(ex\. TensorFlow 2\.9 to TensorFlow 2\.10\) and make them available at the earliest opportunity\. When an existing version of TensorFlow is released with a new version of CUDA, we release a new DLC for that version of TensorFlow with support for the new CUDA version\.

### Will my instance be patched in place while my workload is running?<a name="support-policy-faq-in-place-patch"></a>

No\. Patch updates for DLC are not “in\-place” updates\.

You must delete the existing image on your instance and pull the latest container image without terminating you instance\.

### What happens when a new patched or updated framework version is available?<a name="support-policy-faq-new-image-available"></a>

Regularly check the release notes page for your image\. We encourage you to upgrade to new patched or updated frameworks when they are available\. For a list of available DLCs, see [Release Notes for AWS Deep Learning Containers](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/dlc-release-notes.html)\.

### Are dependencies updated without changing the framework version?<a name="support-policy-faq-dependencies"></a>

We update dependencies without changing the framework version\. However, if a dependency update causes an incompatibility, we create an image with a different version\. Be sure to check the [Release Notes for AWS Deep Learning Containers](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/dlc-release-notes.html) for updated dependency information\.

### When does active support for my framework version end?<a name="support-policy-faq-end-of-support"></a>

DLC images are immutable\. Once they are created they do not change\. There are four main reasons why active support for a framework version ends:
+ [Framework version \(patch\) upgrades](#support-policy-faq-end-of-support-version-patch)
+ [AWS security patches](#support-policy-faq-end-of-support-security-patch)
+ [End of patch date \(Aging out\)](#support-policy-faq-end-of-support-aging-out)
+ [Dependency end\-of\-support](#support-policy-faq-end-of-support-dependency)

**Note**  
Due to the frequency of version patch upgrades and security patches, we recommend checking the release notes page for your DLC often, and upgrading when changes are made\.

#### Framework version \(patch\) upgrades<a name="support-policy-faq-end-of-support-version-patch"></a>

If you have a DLC workload based on TensorFlow 2\.7\.0 and TensorFlow releases version 2\.7\.1 on GitHub, then AWS releases a new DLC with TensorFlow 2\.7\.1\. The previous images with 2\.7\.0 are longer actively maintained once the new image with TensorFlow 2\.7\.1 is released\. The DLC with TensorFlow 2\.7\.0 does not receive further patches\. The DLC release notes page for TensorFlow 2\.7 is then updated with the latest information\. There is no individual release note page for each minor patch\.

New DLCs created due to patch upgrades are designated with updated [release tags](https://github.com/aws/deep-learning-containers/tags)\. If changes are not backwards compatible, the tag will change major versions rather than minor versions \(ex\. v1\.0 will change to v2\.0 rather than v 1\.2\)\.

#### AWS security patches<a name="support-policy-faq-end-of-support-security-patch"></a>

If you have a workload based on an image with TensorFlow 2\.7\.0 and AWS makes a security patch, then a new version of the DLC is released for TensorFlow 2\.7\.0\. The previous version of the images with TensorFlow 2\.7\.0 is no longer actively maintained\. For more information, see [Will my instance be patched in place while my workload is running?](#support-policy-faq-in-place-patch) For steps on finding the latest DLC, see [How can I find the latest patched image for a supported framework version?](#support-policy-faq-latest-patched-image)

New DLCs created due to patch upgrades are designated with updated [release tags](https://github.com/aws/deep-learning-containers/tags)\. If changes are not backwards compatible, the tag will change major versions rather than minor versions \(ex\. v1\.0 will change to v2\.0 rather than v 1\.2\)\.

#### End of patch date \(Aging out\)<a name="support-policy-faq-end-of-support-aging-out"></a>

DLCs hit their end of patch date 365 days after the GitHub release date\. 

**Important**  
We make an exception when there is a major framework update\. For example\. if TensorFlow 1\.15 updates to TensorFlow 2\.0, then we continue to support the most recent version of TensorFlow 1\.15 for a period of two years from the date of the GitHub release or six months after the origin framework maintenance team drops support, whichever date is earlier\.

#### Dependency end\-of\-support<a name="support-policy-faq-end-of-support-dependency"></a>

If you are running a workload on a TensorFlow 2\.7\.0 DLC image with Python 3\.6 and that version of Python is marked for end\-of\-support, then all DLC images based on Python 3\.6 will no longer be actively maintained\. Similarly, if an OS version like Ubuntu 16\.04 is marked for end\-of\-support, then all DLC images that are dependent on Ubuntu 16\.04 will no longer be actively maintained\. 

### Will images with framework versions that are no longer actively maintained be patched?<a name="support-policy-faq-end-of-patch"></a>

No\. Images that are no longer actively maintained will not have new releases\.

### How do I use an older framework version?<a name="support-policy-faq-using-older-framework-version"></a>

To use a DLC with an older framework version, browse the [DLC GitHub release tags ](https://github.com/aws/deep-learning-containers/releases)to find the image URI of your choice and use it to pull the docker image\.

### How do I stay up\-to\-date with support changes in frameworks and their versions?<a name="support-policy-faq-stay-up-to-date"></a>

Stay up\-to\-date with DLC frameworks and versions using the [DLC release notes](https://docs.aws.amazon.com/deep-learning-containers/latest/devguide/dlc-release-notes.html), and the [Available Deep Learning Containers Images](https://github.com/aws/deep-learning-containers/blob/master/available_images.md) page\.

### Do I need a commercial license to use the Anaconda Repository?<a name="support-policy-faq-anaconda-repository"></a>

Anaconda shifted to a commercial licensing model for certain users\. Actively maintained DLCs have been migrated to the publicly available open\-source version of Conda \([conda\-forge](https://anaconda.org/conda-forge)\) from the Anaconda channel\.

**Warning**  
If you are actively using Anaconda to install and manage your packages and their dependencies in a DLC that is no longer actively maintained, you are responsible for complying with the governing license from the [Anaconda Repository,](https://repo.anaconda.com/) if you determine that the terms apply to you\. Alternatively, you can migrate to one of the currently\-supported DLCs listed in the [AWS Deep Learning Containers Framework Support Policy table](https://aws.amazon.com/releasenotes/dlc-support-policy/) or you can install packages using conda\-forge as a source\.