# Multi Process

> Source: `Analyzer Help.chm / 48_MultiProcess.htm`
> Extracted with `extract_chmLib`, decoded as `ks_c_5601-1987` (CP949); HTML converted to Markdown. Text unmodified.

## Multi Process

Purpose

Improves parsing speed

Benefit

If more than 2 log files are merged or model is created by more than 2 batches, parsing speed is 20% faster than single process.

Operating Principle

Divides log file to 2 groups and then performs parsing by groups. After parsing, merges the created models.

Usage of Multi Process

If the Patch for Multi Process is applied, it does not need to additional configuration to use. To support Multi Process, you can select the option as below.

Option Location: tools>options>General>Common

Difference between Single process

Since it performs two processes and each process create separate model, sub model concept is added. If 2 DRM files are merged, two sub models are existed in subfolder of Model.

Two processes are processed in parallel. If the process is not operated for special time, it will be decided to Failure and processes next configured job.

In case of Multi Process is not supported

If the below logs are included, it will be processed as single process.

If the logs which have different file format, for example drm+scn, drm+drmmp, drm+drmw, are merged,

acu/ecu and format log

If you select Process as a file option
