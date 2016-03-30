---
title: Disabling Kerberos Security
---

Follow these steps to disable Kerberos security for HAWQ and PXF for manual installations.

**Note:** If you install or manager your cluster using Ambari, then the HAWQ Ambari plug-in automatically disables security for HAWQ and PXF when you disable security for Hadoop. The following instructions are only necessary for manual installations, or when Hadoop security is disabled outside of Ambari.

1.  Disable Kerberos on the Hadoop cluster on which you use HAWQ.
2.  Disable security for HAWQ:
    1.  Login to the HAWQ database master server as the `gpadmin` user:

        ```
        ssh hawq\_master\_fqdn
        ```

    2.  Run the following commands to set environment variables:

        ```
        $ source /usr/local/hawq/greenplum\_path.sh
        $ export MASTER\_DATA\_DIRECTORY = /gpsql
        ```

        **Note:** Substitute the correct value of MASTER\_DATA\_DIRECTORY for your configuration.

    3.  Start HAWQ if necessary:

        ```
        $ hawq start -a
        ```

    4.  Run the following command to disable security:

        ```
        $ hawq config --masteronly -c enable\_secure\_filesystem -v “off”
        ```

        **Note:** Substitute the correct value of MASTER\_DATA\_DIRECTORY for your configuration.

    5.  Change the permission of the HAWQ HDFS data directory:

        ```
        $ sudo -u hdfs hdfs dfs -chown -R gpadmin:gpadmin /hawq\_data
        ```

    6.  On the HAWQ master node and on all segment server nodes, edit the /usr/local/hawq/etc/hdfs-client.xml file to disable kerberos security. Comment or remove the following properties in each file:

        ```
        <!--
        <property>
          <name>hadoop.security.authentication</name>
          <value>kerberos</value>
        </property>

        <property>
          <name>dfs.namenode.kerberos.principal</name>
          <value>nn/_HOST@LOCAL.DOMAIN</value>
        </property>
        -->
        ```

    7.  Restart HAWQ:

        ```
        $ hawq restart -a -M fast
        ```

3.  Disable security for PXF:
    1.  On each PXF node, edit the /etc/gphd/pxf/conf/pxf-site.xml to comment or remove the properties:

        ```
        <!--
        <property>
            <name>pxf.service.kerberos.keytab</name>
            <value>/etc/security/phd/keytabs/pxf.service.keytab</value>
            <description>path to keytab file owned by pxf service
            with permissions 0400</description>
        </property>

        <property>
            <name>pxf.service.kerberos.principal</name>
            <value>pxf/_HOST@PHD.LOCAL</value>
            <description>Kerberos principal pxf service should use.
            _HOST is replaced automatically with hostnames
            FQDN</description>
        </property>
        -->
        ```

    2.  Restart the PXF service.