# libvirt使用Ceph RBD设备

1. 创建存储池

```bash
ceph osd pool create libvirt-pool 128 128
```

查看创建的存储池

```bash
ceph osd lspools
```

2. 创建Ceph用户```client.libvirt```，设置权限到存储libvirt-pool

```bash
ceph auth get-or-create client.libvirt mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=libvirt-pool'
```

验证：

```bash
ceph auth list
```

显示结果：

```bash
client.libvirt
key: AQBblU1b9FECCRAA4tW8qaBYtxTsDlaNJybZSQ==
caps: [mon] allow r
    caps: [osd] allow class-read object_prefix rbd_children, allow rwx pool=libvirt-pool
```

3. 将虚拟机镜像文件cirros-3.5-x86_64.img导入到存储池libvirt-pool中

```bash
qemu-img convert -f qcow2 -O raw cirros-3.5-x86_64.img rbd:libvirt-pool/cirros3.5.img
```

4. 在libvirt中配置Ceph认证需要的密钥

​        4.1 定义密钥

         ```xml
cat > secret.xml <<EOF
<secret ephemeral='no' private='no'>
    <usage type='ceph'>
        <name>client.libvirt secret</name>
    </usage>
</secret>
EOF
         ```

```bash
virsh secret-define --file secret.xml
```

​       4.2 设置密钥

```bash
virsh secret-list
UUID                                  Usage
---------------------------------------------------------------
 fdcb5967-d3e5-4618-98f5-5919a723e414  ceph client.libvirt secret
```

```bash
virsh secret-set-value --secret fdcb5967-d3e5-4618-98f5-5919a723e414 --base64 AQBblU1b9FECCRAA4tW8qaBYtxTsDlaNJybZSQ==
```

> 其中“AQBblU1b9FECCRAA4tW8qaBYtxTsDlaNJybZSQ==”是从上面ceph auth list的client.libvirt的key字段中得到的

5. 修改虚拟机XML文件的disk部分

```bash
virsh edit test
```

```xml
...
<disk type='network' device='disk'>
      <driver name='qemu'/>
      <auth username='libvirt'>
        <secret type='ceph' uuid='fdcb5967-d3e5-4618-98f5-5919a723e414'/>
      </auth>
      <source protocol='rbd' name='libvirt-pool/centos6864.qcow2.img'>
        <host name='192.168.1.15' port='6789'/>
        <host name='192.168.1.16' port='6789'/>
        <host name='192.168.1.17' port='6789'/>
      </source>
      <target dev='vda' bus='virtio'/>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
</disk>
...
```

6. 验证启动：

   ```bash
   virsh start test --console
   ```


