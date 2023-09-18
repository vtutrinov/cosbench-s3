# cosbench-s3
## Run the cosbench with patched s3 plugin
1. Start the cosbench container from openio/cosbench-openio.
```bash
docker run -dit --name cosbench --net=host  -e CONTROLLER=true -e DRIVER=true -e COSBENCH_PLUGINS="S3" -v ./:/cosbench-s3-plugin openio/cosbench-openio /bin/bash
```
2. Attach to the created container
```bash
docker attach cosbench
```

3. Edit the script to run cosbench

```bash
vim start-cosbench.sh

#replace
# 'S3')      COSBENCH_OSGI="$COSBENCH_OSGI"' cosbench-s3_${VERSION}' ;;
# by 
# 'S3')      COSBENCH_OSGI="$COSBENCH_OSGI"' cosbench-s3_0.4.3.0' ;;
```
4. Copy the cosbench-s3 osgi plugin from this repo
```bash
cp /cosbench-s3-plugin/cosbench-s3_0.4.3.0.jar ./osgi/plugins/
```
5. Start the cosbench
```bash
./start-cosbench.sh
```
## Workload

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<workload name="s3-sample" description="sample benchmark for s3">

    <storage type="s3" config="timeout=800000;accessKey=<awsAccessKey>;secretKey=<awsSecret>;endpoint=<s3_endpoint>;pathStyleAccess=true;signerOverride=AWS4UnsignedPayloadSignerType" />

    <workflow>

        <workstage name="init">
            <work type="init" workers="1" config="cprefix=bucket1;containers=r(1,2)" />
        </workstage>

        <workstage name="prepare">
            <work type="prepare" workers="1" config="cprefix=bucket1;containers=r(1,2);objects=r(1,10);sizes=c(64)KB" />
        </workstage>

        <workstage name="main">
            <work name="main" workers="8" runtime="30">
                <operation type="read" ratio="80" config="cprefix=bucket1;containers=u(1,2);objects=u(1,10)" />
                <operation type="write" ratio="20" config="cprefix=bucket1;containers=u(1,2);objects=u(11,20);sizes=c(64)KB" />
            </work>
        </workstage>

        <workstage name="cleanup">
            <work type="cleanup" workers="1" config="cprefix=bucket1;containers=r(1,2);objects=r(1,20)" />
        </workstage>

        <workstage name="dispose">
            <work type="dispose" workers="1" config="cprefix=bucket1;containers=r(1,2)" />
        </workstage>
    </workflow>

</workload>
```
