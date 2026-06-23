# Long-Term Preservation of Research Data

## Why should research data be preserved?

There are several reasons. On the one hand, research data should be preserved to make the results
reproducible. On the other hand research data could be used a second time for investigating another
question. In the latter case persistent identifiers (like DOI) are needed to make these data
findable and citable. In both cases it is important to add meta-data to the data.

## Which research data should be preserved?

Since large quantities of data are nowadays produced it is not possible to store everything. The
researcher needs to decide which data are worth and important to keep.

In case these data come from simulations, there are two possibilities: 1 Storing the result of the
simulations 1 Storing the software and the input-values

Which of these possibilities is preferable depends on the time the simulations need and on the size
of the result of the calculations. Here one needs to estimate, which possibility is cheaper.

**This is, what DFG says** (translated from
<http://www.dfg.de/download/pdf/foerderung/programme/lis/ua_inf_empfehlungen_200901.pdf>, page 2):

*Primary research data are data, which were created in the course* *of studies of sources,
experiments, measurements or surveys. They are the* *basis of scholarly publications*. *The
definition of primary research data depends on the subject*. *Each community of researchers should
decide by itself, if raw data are* *already primary research data or at which degree of aggregation
data* *should be preserved. Further it should be agreed upon the granularity* *of research data: how
many data yield one set of data, which will be* *given a persistent identifier*.

## Why should I add Meta-Data to my data?

Many researchers think, that adding meta-data is time-consuming and senseless but that isn't true.
On the contrary, adding meta-data is very important, since they should enable other researchers to
know, how and in which circumstances these data are created, in which format they are saved, and
which software in which version is needed to view the data, and so on, so that other researchers can
reproduce these data or use them for new investigations. Last but not least meta-data should enable
you in ten years time to know what your data describe, which you created such a long time ago.

## What are Meta-Data?

Meta-data means data about data. Meta-data are information about the stored file. There can be
administrative meta-data, descriptive meta-data, technical meta-data and so on. Often meta-data are
stored in XML-format but free text is also possible. There are some meta-data standards like
[Dublin Core](http://dublincore.org/) or
[LMER](https://www.dnb.de/DE/Professionell/Standardisierung/Standards/_content/lmer_uof.html)
Below are some examples:

- possible meta-data for a book would be:
    - Title
    - Author
    - Publisher
    - Publication
    - year
    - ISBN
- possible meta-data for an electronically saved image would be:
    - resolution of the image
    - information about the color depth of the picture
    - file format (jpg or tiff or ...)
    - file size how was this image created (digital camera, scanner, ...)
    - description of what the image shows
    - creation date of the picture
    - name of the person who made the picture
- meta-data for the result of a calculation/simulation could be:
    - file format
    - file size
    - input data
    - which software in which version was used to calculate the result/to do the simulation
    - configuration of the software
    - date of the calculation/simulation (start/end or start/duration)
    - computer on which the calculation/simulation was done
    - name of the person who submitted the calculation/simulation
    - description of what was calculated/simulated

## Where can I get more information about management of research data?

Please visit the wiki [forschungsdaten.org](https://www.forschungsdaten.org/en/) to learn more about
all of the different aspects of research data management.

For questions or individual consultations regarding research data management in general or any of
its certain aspects, you can contact the
[Service Center Research Data](https://tu-dresden.de/forschung-transfer/services-fuer-forschende/kontaktstelle-forschungsdaten?set_language=en)
(Kontaktstelle Forschungsdaten) of TU Dresden.

## I want to archive my research data at ZIH safely. How can I do that?

For TU Dresden there exist two different services at ZIH for archiving research data. Both of
them ensure high data safety by duplicating data internally at two separate locations and
require some data preparation (e.g. packaging), but serve different use cases:

### Storing very infrequently used data during the course of the project

The intermediate archive is a tape storage easily accessible as a directory
(`/archiv/<HRSK-project>/` or `/archiv/<login>/`) using the
[Dataport Nodes](../data_transfer/dataport_nodes.md)
and
[Datamover tools](../data_transfer/datamover.md) to move your data to.
For detailed information please visit the
[ZIH intermediate archive documentation](https://tu-dresden.de/zih/dienste/service-katalog/arbeitsumgebung/backup_archiv/archivierung_am_zih#section-2-1).

!!! note

    The usage of the HRSK-project-related archive is preferable to the login-related archive, as
    this enables assigning access rights and responsibility across multiple researchers, due to the
    common staff turnover in research.

The use of the intermediate archive usually is limited by the end of the corresponding
research project. Afterwards data is required to be removed, tidied up and submitted to a
long-term repository (see next section).

The intermediate archive is the preferred service when you keep large, mostly unused data volumes
during the course of your research project; if you want or need to free storage capacities, but
you are still not able to define certain or relevant datasets for long-term archival.

If you are able to identify complete and final datasets, which you probably won't use actively
anymore, then repositories as described in the next section may be the more appropriate selection.

### Archiving data beyond the project lifetime, for 10 years and above

According to good scientific practice (cf.
[DFG guidelines, #17](https://www.dfg.de/download/pdf/foerderung/rechtliche_rahmenbedingungen/gute_wissenschaftliche_praxis/kodex_gwp.pdf))
and
[TU Dresden research data guidelines](https://tu-dresden.de/tu-dresden/qualitaetsmanagement/ressourcen/dateien/wisprax/Leitlinien-fuer-den-Umgang-mit-Forschungsdaten-an-der-TU-Dresden.pdf),
relevant research data needs to be archived at least for 10 years. The
[OpARA service](https://opara.zih.tu-dresden.de/xmlui/) (Open Access Repository and Archive) is the
joint research data repository service for Saxon universities to address this requirement.

Data can be uploaded and, to comply to the demands of long-term understanding of data, additional
metadata and description must be added. Large datasets may be optionally imported beforehand. In
this case, please contact the
[TU Dresden Service Desk](mailto:servicedesk@tu-dresden.de?subject=OpARA:%20Data%20Import).
Optionally, data can also be **published** by OpARA. To ensure data quality, data submissions
undergo a review process.

Beyond OpARA, it is also recommended to use discipline-specific data repositories for data
publications. Usually those are well known in a scientific community, and offer better fitting
options of data description and classification. Please visit [re3data.org](https://re3data.org)
to look up a suitable one for your discipline.
