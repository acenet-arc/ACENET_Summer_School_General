---
title: "Input and Output"
teaching: 12
exercises: 3
objectives:
- Sketch the storage structure of a generic, typical cluster
- Understand the terms "local disk", "IOPS", "SAN", "metadata", "parallel I/O"
---

## Cluster storage

Recall what you learned about cluster architecture in the Intro to ACENET, or
look at the video here: [https://www.youtube.com/watch?v=VxmTbDfelmA](https://www.youtube.com/watch?v=VxmTbDfelmA)
- There is a login node (or maybe more than one). 
- There might be a data transfer node (DTN), which is basically a login node
specially designated for doing (!) data transfers.
- There are a lot of compute nodes.  The compute nodes may or may not have local disk.
- Most I/O goes to a storage array or SAN (Storage Array Network).

Broadly, you have two choices: You can do I/O to the node-local disk (if there is any),
or you can do I/O to the SAN.  Local disk suffers little or no contention, but is inconvenient.

> ## Local disk
> What are the inconveniences of local disk? What sort of work patterns suffer most from
> this? What suffers least? That is, what sort of jobs can use local disk most easily?
{: .challenge}

Local disk performance depends on a lot of things. If you're interested you can get an idea from here: 
[What-is-the-throughput-of-15k-rpm-sas-drive](https://serverfault.com/questions/190451/what-is-the-throughput-of-15k-rpm-sas-drive)
- For *that particular* disk model, throughput 16MB/s to 200MB/s depending on how you do it
- IOPS (IO operations per second) 150-200

Most input and output on a cluster goes through the SAN.

There are many architectural choices made in constructing a SAN.
- What technology? Lustre popular these days. ACENET & Compute Canada use it. GPFS is another...
- How many servers, and how many disks in each? What disks?
- How many MDS? MDS = MetaData Server. Things like 'ls' only require *metadata*. Exactly what is handled by the MDS (or even if there is one) may depend on the technology chosen (e.g. Lustre).
- What switches and how many of them? 
- Are things wired together with fibre or with ethernet? What's the wiring topology?
- Where is there redundancy or failovers, and how much?
This is all the domain of the sysadmins, but what should you as the user do about it?

If you're doing parallel computing you have further  choices about how you do that.

- _File-per-process_ is reliable, portable and simple, but lousy for checkpointing and restarting.

- Many small files on a SAN (or anywhere, really) leads to metadata bottlenecks.
Most HPC filesystems assume no more than 1,000 files per directory.

- Parallel I/O (e.g. MPI-IO) -> many processes, one file
- Solves restart problems, but requires s/w infrastructure and programming

- High-level interfaces like NetCDF and HDF5 are highly recommended
- I/O bottlenecks:
- Disk read-write rate. Alleviated by striping.
- Fibre bandwidth (or Ethernet or ...)
- Switch capacity.
- Metadata accesses (especially if only one MDS)

> ## Measuring I/O rates
>
> - Look into IOR on [GitHub](https://github.com/LLNL/ior) or [SourceForge](https://sourceforge.net/projects/ior-sio/)
> - Read this 2007 paper, [Using IOR to Analyze the I/O performance for HPC Platforms](https://cug.org/5-publications/proceedings_attendee_lists/2007CD/S07_Proceedings/pages/Authors/Shan/Shan_paper.pdf)
> - Neither of these are things to do right this minute!
{: .challenge}

--- Takeaways:
- I/O is complicated. If you really want to know what performs best, you'll have to experiment.
- A few large IO operations are better than many small ones.
- A few large files are usually better than many small files.
- High-level interfaces (e.g. HDF5) are your friend, but even they're not perfect.

## Moving data on and off a cluster

- "Internet" cabling varies a lot. 100Mbit/s widespread, 1Gbit becoming common, 10Gbit or more between *most* CC sites
- Canarie is the Cdn research network. Check out their ["weather map"](http://weathermap.canarie.ca/index.html)
- Firewalls sometimes the problem - security versus speed

- Filesystem at sending or receiving end often the bottleneck (SAN or simple disk)
- What are typical disk I/O rates? See above! Highly variable! 

> ## Estimating transfer times
> 1. How long to move a gigabyte at 100Mbit/s?
> 2. How long to move a terabyte at 1Gbit/s? 
>
> > ## Solution
> > Remember a byte is 8 bits, a megabyte is 8 megabits, etc.
> > 1. 1024MByte/GByte * 8Mbit/MByte / 100Mbit/sec = 81 sec
> > 2. 8192 sec = 2 hours and 20 minutes
> > 
> > Remember these are "theoretical maximums"! There is almost always some other bottleneck or contention that reduces this!
> >
> {: .solution}
{: .challenge}

- Restartable downloads (wget? rsync? Globus!)
- checksumming (md5sum) to verify integrity of large transfers

