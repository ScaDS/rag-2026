# User Support

## Ask the HPC Buddy

Our colleagues from ScaDS.AI have created the [HPC Buddy](https://assistant.llm.scads.ai/HPC-Buddy),
an AI that tries to answer your questions using knowledge from this documentation. It is updated
regularly, but not always accurate, as it has no real understanding of this documentation. However,
it provides sources, so you may try it out before you consider the steps mentioned below.

On all clusters, you can also use the script `ask_assistant.sh` to send questions to the HPC Buddy.

```console
marie@login$ ask_assistant.sh
Your request to HPC-Buddy: How many GPUs has Capella?

Asking HPC-Buddy...

Answer:

According to the information provided, each node of the Capella cluster has **4 x NVIDIA H100-SXM5 Tensor Core-GPUs**. The cluster consists of **156 nodes**.

Source: https://compendium.hpc.tu-dresden.de/jobs_and_resources/hardware_overview/#capella
Source: https://compendium.hpc.tu-dresden.de/jobs_and_resources/capella/#overview

Therefore, the total number of GPUs in the Capella cluster is **156 nodes * 4 GPUs per node = 624 GPUs**.

Your request to HPC-Buddy:
```

!!! warning "Correctness"

    Note that the HPC Buddy does not really understand your requests. While it tries to give you
    the sources for its statements, it is not always correct even when it sounds convincing.
    The HPC Buddy is configured to provide links to this documentation, so that you can follow them
    to check whether information provided by the HPC Buddy is correct.

## Create a Ticket

The best way to ask for help: send a message to
[hpc-support@tu-dresden.de](mailto:hpc-support@tu-dresden.de) with a
detailed description of your problem.

It should include:

- Who is reporting? (login name)
- Where have you seen the problem? (name of the HPC system and/or of the node)
- When has the issue occurred? Maybe, when did it work last?
- What exactly happened?

If possible include

- job ID,
- batch script,
- filesystem path,
- loaded modules and environment,
- output and error logs,
- steps to reproduce the error.

This email automatically opens a trouble ticket which will be tracked by the HPC team. Please
always keep the ticket number in the subject on your answers so that our system can keep track
on our communication.

For a new request, please simply send a new email (without any ticket number).

!!! hint "Please try to find an answer in this documentation first."

## Open Q&A Sessions

We invite you to join our public Q&A sessions with any questions you may have:

- Open Q&A session for users of the NHR@TUD HPC systems.
     - Every Monday from 1:30 - 2:00 pm, except on holidays.
     - See [ZIH calendar](https://tu-dresden.de/zih/das-department/termine/termine)
       for the next event and to download the event series to your calendar.
     - Enter the
       [virtual HPC Q&A session room](https://bbb.tu-dresden.de/rooms/qoh-djc-qo9-2wy/join).

- Open AI Q&A session as a joint initiative for users of NHR and GCS centers.
     - Every Thursday from 2:00 - 3:00 pm.
     - See [NHR - AI on High Performance Computers](https://www.nhr-verein.de/science-network/ai-ml/)
       for further details.
     - Enter the [virtual AI Q&A session room](https://www.nhr-verein.de/en/ai-supercomputers).
