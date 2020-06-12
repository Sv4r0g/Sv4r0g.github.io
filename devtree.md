## ���� ���� � devicetree, DTB, DTS

��� ���������� �������� ��������� � ������� `binwalk` ������� ����� ��������� ��������� *Flattened device tree*. ��� ����� ��������� ��� ��� �����, ��������� � �� �����, ����� � ������� ��� ������� ���-�� ���-�� ������� ���� ������. ��������, ��� �� ������������ �������� ��� ������ ��������, �� � �� ��� ���������� �� �������� �������� �������������. � ���� ������� �� ��������� ��� �� ���������� �� ����� ����� ������� �� ������� ����� �� ��������.

```bash
kalosof@ubuntu:~/rev/ui/UniFi_Dream_Machine$ binwalk --term 89cd-udmbase-1.5.6-7117e472b4994c3d9d8c64d053ca3e69.bin 

DECIMAL       HEXADECIMAL     DESCRIPTION
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
0             0x0             Ubiquiti firmware header, header size: 264 bytes, ~CRC32: 0x3B493E07, version: "UDM.alpinev2.v1.5.6.615c4d9.200327.1930"
637           0x27D           Flattened device tree, size: 4710814 bytes, version: 17
881           0x371           gzip compressed data, has original file name: "Image", from Unix, last modified: 2020-03-27 20:36:11
4505429       0x44BF55        Flattened device tree, size: 23666 bytes, version: 17
4529733       0x451E45        Flattened device tree, size: 25607 bytes, version: 17
4555973       0x4584C5        Flattened device tree, size: 23654 bytes, version: 17
4580265       0x45E3A9        Flattened device tree, size: 25248 bytes, version: 17
4606149       0x4648C5        Flattened device tree, size: 25242 bytes, version: 17
4632025       0x46ADD9        Flattened device tree, size: 23637 bytes, version: 17
4656301       0x470CAD        Flattened device tree, size: 25582 bytes, version: 17
4682521       0x477319        Flattened device tree, size: 24472 bytes, version: 17
```

### �����������

�������� [������������](https://github.com/devicetree-org/devicetree-specification/releases/tag/v0.3) devicetree - ��� ��������� ������, ������� ������ ��������� ���������� ������������ � ������������ ����������� ������������. ��������, ��� �������� ������������ �������.

### DTB

� �������� �� ����� ���� � �������� �������������� ��������� *devicetree* - *Device Tree Blob (DTB)*.
��� ������ ������, ���� ��������� �������� ������ ���� �� ��� �� ��������.

��������� DTB �� �������� � ����� ���� � ��������, ��� ������ �� ����������:

```bash
kalosof@ubuntu:~/rev/ui/UniFi_Dream_Machine$ dd if=89cd-udmbase-1.5.6-7117e472b4994c3d9d8c64d053ca3e69.bin skip=$((0x44BF55)) count=$((0x477319 - 0x44BF55)) bs=1 of=devtree.dtb

kalosof@ubuntu:~/rev/ui/UniFi_Dream_Machine/devtree$ file devtree.dtb
devtree.dtb: Device Tree Blob version 17, size=23666, boot CPU=0, string block size=1474, DT structure block size=22120

```

��������� DTB ����������� ���������� *fdt_header* � ����� ���:

```c
struct fdt_header {
	uint32_t magic;
	uint32_t totalsize;
	uint32_t off_dt_struct;
	uint32_t off_dt_strings;
	uint32_t off_mem_rsvmap;
	uint32_t version;
	uint32_t last_comp_version;
	uint32_t boot_cpuid_phys;
	uint32_t size_dt_strings;
	uint32_t size_dt_struct;
};
```

������ ���� ��������� ����� ������ 4 ����� � ������� big-endian.
![dtb1](resources/dtb1.png)

� �� ���� ������������� ������ ���� �� �����������, �.�. � ���� ��� ������ �������������. ��������� � DTB ����� ��������� � ������������. ��� ������������� ��������� - ��� ����������� ������������ ���� ������ � �������� ���.

### DTS

*DTS (Device Tree Source)* - ��� ��������� ������������� DTB, ������� ���� �� ����� �� ������������� ������ ��������� ��������.

��� ������������ ����������� device-tree-decompiler:

```bash
kalosof@ubuntu:~/rev/ui/UniFi_Dream_Machine/devtree$ sudo snap install device-tree-decompiler
kalosof@ubuntu:~/rev/ui/UniFi_Dream_Machine/devtree$ dtc -I dts -O dtb -o devtree.dtb system.dts
kalosof@ubuntu:~/rev/ui/UniFi_Dream_Machine/devtree$ file system.dts
system.dts: Device Tree File (v1), ASCII text, with very long lines
```

![dts1](resources/dts1.png)

DTS ������ ����� ��� ������, �������� ���������, ���� � ��������, ������������ �������������. ��� devicetree ������ ����������� ����� ���� */cpus* � */memory*.

���� ����� �������� ���������� �� ��������� �����, �� �� ����� ����� ����� ���������� ����������, ��������:
* ������� ���������� ����������� - UART, JTAG.
```c
 uart0 {
    compatible = "ns16550a";
    device_type = "serial";
    reg = <0x00 0xfd883000 0x00 0x1000>;
    clock-frequency = <0x00>;
    interrupts = <0x00 0x11 0x04>;
    reg-shift = <0x02>;
    reg-io-width = <0x04>;
};
...
```
* �������� ��������� ����-������, ��� ����� ���� ������� ��� ������ ����� �������� �������������� � ����������� ����������� ������� ����.
```c
 nand-flash {
        compatible = "annapurna-labs,al-nand";
        reg = <0x00 0xfa100000 0x00 0x202000>;
        interrupts = <0x00 0x01 0x04>;
        clocks = <0x08>;
        clock-names = "sbclk";
        #address-cells = <0x01>;
        #size-cells = <0x01>;
        max-onfi-timing-mode = <0x01>;
        status = "disabled";
  
        partition@0 {
                label = "al_boot";
                reg = <0x00 0x200000>;
        };
  
        partition@1 {
                label = "device_tree";
                reg = <0x200000 0x100000>;
        };
  
        partition@2 {
                label = "linux_kernel";
                reg = <0x300000 0x1300000>;
        };
  
        partition@3 {
                label = "ubifs";
                reg = <0x1300000 0x1e600000>;
        };
};
spi {
            compatible = "snps,dw-spi-mmio\0snps,dw-apb-ssi";
            #address-cells = <0x01>;
            #size-cells = <0x00>;
            reg = <0x00 0xfd882000 0x00 0x1000>;
            interrupts = <0x00 0x17 0x04>;
            num-chipselect = <0x04>;
            bus-num = <0x00>;
            clocks = <0x08>;
            clock-names = "sbclk";
  
            spiflash@0 {
                    #address-cells = <0x01>;
                    #size-cells = <0x01>;
                    compatible = "spi_flash_jedec_detection";
                    spi-max-frequency = <0x23c3460>;
                    reg = <0x00>;
  
                    partition@0 {
                            reg = <0x00 0x1c0000>;
                            label = "u-boot";
                    };
  
                    partition@1 {
                            reg = <0x1c0000 0x10000>;
                            label = "u-boot-env";
                    };
                partition@2 {
                        reg = <0x1d0000 0x10000>;
                        label = "u-boot-env-2";
                };

                partition@3 {
                        reg = <0x1e0000 0x10000>;
                        label = "Factory";
                    };

                partition@4 {
                            reg = <0x1f0000 0x10000>;
                        label = "EEPROM";
                };
        };
};


```

� �����, dts-���� �������� ����� ��������� ���������� � ������� ���������� � ��� ����������� � ����� ������������ ������� ��� ������������ ��������, ������ ������������� ����������� � �������� (��������, ������������� ������������), ����������� �������� ������������ � ������ �����������, ��������� � ������������ ������� ������ � ������� ��� ����� �����������������. ��� ��� �����, ����� �������� ������ ������.
