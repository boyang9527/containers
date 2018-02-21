---

copyright:
  years: 2014, 2018
lastupdated: "2018-01-12"

---

{:new_window: target="_blank"}
{:shortdesc: .shortdesc}
{:screen: .screen}
{:pre: .pre}
{:table: .aria-labeledby="caption"}
{:codeblock: .codeblock}
{:tip: .tip}
{:download: .download}


# 配置叢集的子網路
{: #subnets}

將子網路新增至叢集，以變更可用的可攜式公用或專用 IP 位址的儲存區。
{:shortdesc}

在 {{site.data.keyword.containershort_notm}} 中，您可以將網路子網路新增至叢集，為 Kubernetes 服務新增穩定的可攜式 IP。在此情況下，子網路不會與網路遮罩搭配使用，以在一個以上的叢集中建立連線功能。而是使用子網路，從可用來存取該服務的叢集中提供服務的永久固定 IP。

當您建立標準叢集時，{{site.data.keyword.containershort_notm}} 會自動佈建 1 個可攜式公用子網路（含 5 個公用 IP 位址）及 1 個可攜式專用子網路（含 5 個專用 IP 位址）。可攜式公用及專用 IP 位址是靜態的，不會在移除工作者節點或甚至叢集時變更。針對每一個子網路，將其中一個可攜式公用 IP 位址及其中一個可攜式專用 IP 位址用於[應用程式負載平衡器](cs_ingress.html)，您可以使用應用程式負載平衡器來公開叢集中的多個應用程式。藉由[建立負載平衡器服務](cs_loadbalancer.html)，即可使用其餘 4 個可攜式公用 IP 位址及 4 個可攜式專用 IP 位址，將單一應用程式公開給大眾使用。

**附註：**可攜式公用 IP 位址是按月收費。如果您選擇在佈建叢集之後移除可攜式公用 IP 位址，則仍需要按月支付費用，即使您僅短時間使用。

## 要求叢集的其他子網路
{: #request}

您可以藉由將子網路指派給叢集，來為叢集新增穩定的可攜式公用或專用 IP。

**附註：**當您讓叢集可以使用子網路時，會使用這個子網路的 IP 位址來進行叢集網路連線。若要避免 IP 位址衝突，請確定您使用的子網路只有一個叢集。請不要同時將子網路用於多個叢集或 {{site.data.keyword.containershort_notm}} 以外的其他用途。

開始之前，請先將 [CLI 的目標](cs_cli_install.html#cs_cli_configure)設為您的叢集。

若要在 IBM Cloud 基礎架構 (SoftLayer) 帳戶中建立子網路，並將它設為可供指定的叢集使用，請執行下列動作：

1. 佈建新的子網路。

    ```
    bx cs cluster-subnet-create <cluster_name_or_id> <subnet_size> <VLAN_ID>
    ```
    {: pre}

    <table>
    <caption>瞭解此指令的元件</caption>
    <thead>
    <th colspan=2><img src="images/idea.png" alt="構想圖示"/> 瞭解此指令的元件</th>
    </thead>
    <tbody>
    <tr>
    <td><code>cluster-subnet-create</code></td>
    <td>佈建叢集子網路的指令。</td>
    </tr>
    <tr>
    <td><code><em>&lt;cluster_name_or_id&gt;</em></code></td>
    <td>將 <code>&gt;cluster_name_or_id&lt;</code> 取代為叢集的名稱或 ID。</td>
    </tr>
    <tr>
    <td><code><em>&lt;subnet_size&gt;</em></code></td>
    <td>將 <code>&gt;subnet_size&lt;</code> 取代為您要從可攜式子網路新增的 IP 位址數目。接受值為 8、16、32 或 64。<p>**附註：**當您新增子網路的可攜式 IP 位址時，會使用三個 IP 位址來建立叢集內部網路，因此，您無法將它們用於應用程式負載平衡器，或是使用它們來建立負載平衡器服務。例如，如果您要求八個可攜式公用 IP 位址，則可以使用其中的五個將您的應用程式公開給大眾使用。</p> </td>
    </tr>
    <tr>
    <td><code><em>&lt;VLAN_ID&gt;</em></code></td>
    <td>將 <code>&gt;VLAN_ID&lt;</code> 取代為您要在其上配置可攜式公用或專用 IP 位址之公用或專用 VLAN 的 ID。您必須選取現有工作者節點所連接的公用或專用 VLAN。若要檢閱工作者節點的公用或專用 VLAN，請執行 <code>bx cs worker-get &gt;worker_id&lt;</code> 指令。</td>
    </tr>
    </tbody></table>

2.  驗證已順利建立子網路並將其新增至叢集。子網路 CIDR 列在 **VLAN** 區段中。

    ```
    bx cs cluster-get --showResources <cluster name or id>
    ```
    {: pre}

<br />


## 將自訂及現有子網路新增至 Kubernetes 叢集
{: #custom}

您可以將現有可攜式公用或專用子網路新增至 Kubernetes 叢集。

開始之前，請先將 [CLI 的目標](cs_cli_install.html#cs_cli_configure)設為您的叢集。

如果 IBM Cloud 基礎架構 (SoftLayer) 組合中的現有子網路具有自訂防火牆規則或您要使用的可用 IP 位址，則請建立沒有子網路的叢集，並在叢集佈建時，將現有子網路設為可供叢集使用。

1.  識別要使用的子網路。請記下子網路的 ID 及 VLAN ID。在此範例中，子網路 ID 是 807861，而 VLAN ID 是 1901230。

    ```
    bx cs subnets
    ```
    {: pre}

    ```
    Getting subnet list...
    OK
    ID        Network                                      Gateway                                   VLAN ID   Type      Bound Cluster   
    553242    203.0.113.0/24                               10.87.15.00                               1565280   private      
    807861    192.0.2.0/24                                 10.121.167.180                            1901230   public      
    
    ```
    {: screen}

2.  確認 VLAN 所在的位置。在此範例中，位置是 dal10。

    ```
    bx cs vlans dal10
    ```
    {: pre}

    ```
    Getting VLAN list...
    OK
    ID        Name                  Number   Type      Router   
    1900403   vlan                    1391     private   bcr01a.dal10   
    1901230   vlan                    1180     public   fcr02a.dal10 
    ```
    {: screen}

3.  使用您所識別的位置及 VLAN ID 來建立叢集。包括 `--no-subnet` 旗標，以防止自動建立新的可攜式公用 IP 子網路及新的可攜式專用 IP 子網路。

    ```
    bx cs cluster-create --location dal10 --machine-type u2c.2x4 --no-subnet --public-vlan 1901230 --private-vlan 1900403 --workers 3 --name my_cluster
    ```
    {: pre}

4.  驗證已要求建立叢集。

    ```
    bx cs clusters
    ```
    {: pre}

    **附註：**訂購工作者節點機器，並在您的帳戶中設定及佈建叢集，最多可能需要 15 分鐘。

    叢集佈建完成之後，叢集的狀態會變更為 **deployed**。

    ```
    Name         ID                                   State      Created          Workers   
    my_cluster   paf97e8843e29941b49c598f516de72101   deployed   20170201162433   3   
    ```
    {: screen}

5.  檢查工作者節點的狀態。

    ```
    bx cs workers <cluster>
    ```
    {: pre}

    工作者節點備妥後，狀態會變更為 **Normal**，而且狀態為 **Ready**。當節點狀態為 **Ready** 時，您就可以存取叢集。

    ```
    ID                                                  Public IP        Private IP     Machine Type   State      Status  
    prod-dal10-pa8dfcc5223804439c87489886dbbc9c07-w1   169.47.223.113   10.171.42.93   free           normal    Ready
    ```
    {: screen}

6.  指定子網路 ID，以將子網路新增至叢集。當您讓叢集可以使用子網路時，會自動建立 Kubernetes 配置對映，其中包括您可以使用的所有可用的可攜式公用 IP 位址。如果您的叢集還沒有應用程式負載平衡器，則會自動使用一個可攜式公用 IP 位址及一個可攜式專用 IP 位址來建立公用及專用應用程式負載平衡器。所有其他可攜式公用或專用 IP 位址，則可用來為您的應用程式建立負載平衡器服務。

    ```
    bx cs cluster-subnet-add mycluster 807861
    ```
    {: pre}

<br />


## 將使用者管理的子網路及 IP 位址新增至 Kubernetes 叢集
{: #user_managed}

從您想要 {{site.data.keyword.containershort_notm}} 存取的內部部署網路，提供您自己的子網路。然後，您可以將來自該個子網路的專用 IP 位址，新增到 Kubernetes 叢集中的負載平衡器服務。

需求：
- 使用者管理的子網路只能新增至專用 VLAN。
- 子網路字首長度限制為 /24 到 /30。例如，`203.0.113.0/24` 指定 253 個可用的專用 IP 位址，而 `203.0.113.0/30` 指定 1 個可用的專用 IP 位址。
- 子網路中的第一個 IP 位址必須用來作為子網路的閘道。

開始之前：
- 配置進出外部子網路的網路資料流量遞送。
- 確認您在內部部署資料中心閘道裝置與 IBM Cloud 基礎架構 (SoftLayer) 組合中的專用網路 Vyatta 或叢集裡執行的 Strongswan VPN 服務之間，具有 VPN 連線功能。若要使用 Vyatta，請參閱此[部落格文章 ![外部鏈結圖示](../icons/launch-glyph.svg "外部鏈結圖示")](https://www.ibm.com/blogs/bluemix/2017/07/kubernetes-and-bluemix-container-based-workloads-part4/)。若要使用 Strongswan，請參閱[使用 Strongswan IPSec VPN 服務設定 VPN 連線功能](cs_vpn.html)。

1. 檢視叢集「專用 VLAN」的 ID。找出 **VLAN** 區段。在 **User-managed** 欄位中，將 VLAN ID 識別為 _false_。

    ```
    bx cs cluster-get --showResources <cluster_name>
    ```
    {: pre}

    ```
    VLANs
    VLAN ID   Subnet CIDR         Public       User-managed
    1555503   192.0.2.0/24        true         false
    1555505   198.51.100.0/24     false        false
    ```
    {: screen}

2. 將外部子網路新增至您的專用 VLAN。可攜式專用 IP 位址會新增至叢集的配置對映。

    ```
    bx cs cluster-user-subnet-add <cluster_name> <subnet_CIDR> <VLAN_ID>
    ```
    {: pre}

    範例：

    ```
    bx cs cluster-user-subnet-add my_cluster 203.0.113.0/24 1555505
    ```
    {: pre}

3. 驗證已新增使用者提供的子網路。**User-managed** 欄位為 _true_。

    ```
    bx cs cluster-get --showResources <cluster_name>
    ```
    {: pre}

    ```
    VLANs
    VLAN ID   Subnet CIDR         Public       User-managed
    1555503   192.0.2.0/24        true         false
    1555505   198.51.100.0/24     false        false
    1555505   203.0.113.0/24      false        true
    ```
    {: screen}

4. 新增專用負載平衡器服務或專用 Ingress 應用程式負載平衡器，以透過專用網路存取您的應用程式。如果您要使用來自建立專用負載平衡器或專用 Ingress 應用程式負載平衡器時所新增之子網路的專用 IP 位址，則必須指定 IP 位址。否則，會從 IBM Cloud 基礎架構 (SoftLayer) 子網路或專用 VLAN 上使用者提供的子網路中，隨機選擇一個 IP 位址。如需相關資訊，請參閱[使用負載平衡器服務類型來配置應用程式的存取](cs_loadbalancer.html#config)或[啟用專用應用程式負載平衡器](cs_ingress.html#private_ingress)。

<br />


## 管理 IP 位址及子網路
{: #manage}

您可以使用可攜式公用及專用子網路及 IP 位址來公開叢集中的應用程式，並且將它們設為可從網際網路或在專用網路上進行存取。
{:shortdesc}

在 {{site.data.keyword.containershort_notm}} 中，您可以將網路子網路新增至叢集，為 Kubernetes 服務新增穩定的可攜式 IP。當您建立標準叢集時，{{site.data.keyword.containershort_notm}} 會自動佈建 1 個可攜式公用子網路（含 5 個可攜式公用 IP 位址）及 1 個可攜式專用子網路（含 5 個可攜式專用 IP 位址）。可攜式 IP 位址是靜態的，不會在移除工作者節點或甚至叢集時變更。

兩個可攜式 IP 位址（一個公用、一個專用）會用於 [Ingress 應用程式負載平衡器](cs_ingress.html)，您可以使用 Ingress 應用程式負載平衡器來公開叢集中的多個應用程式。4 個可攜式公用和 4 個可攜式專用 IP 位址可用來藉由[建立負載平衡器服務](cs_loadbalancer.html)來公開應用程式。

**附註：**可攜式公用 IP 位址是按月收費。如果您選擇在佈建叢集之後移除可攜式公用 IP 位址，則仍需要按月支付費用，即使您僅短時間使用。



1.  建立名為 `myservice.yaml` 的 Kubernetes 服務配置檔，並且使用虛擬負載平衡器 IP 位址來定義類型為 `LoadBalancer` 的服務。下列範例使用 IP 位址 1.1.1.1 作為負載平衡器 IP 位址。

    ```
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        run: myservice
      name: myservice
      namespace: default
    spec:
      ports:
      - port: 80
        protocol: TCP
        targetPort: 80
      selector:
        run: myservice
      sessionAffinity: None
      type: LoadBalancer
      loadBalancerIP: 1.1.1.1
    ```
    {: codeblock}

2.  在叢集中建立服務。

    ```
    kubectl apply -f myservice.yaml
    ```
    {: pre}

3.  檢查服務。

    ```
    kubectl describe service myservice
    ```
    {: pre}

    **附註：**建立此服務失敗，因為 Kubernetes 主節點在 Kubernetes 配置對映中找不到指定的負載平衡器 IP 位址。當您執行此指令時，可以看到錯誤訊息以及叢集可用的公用 IP 位址清單。

    ```
    Error on cloud load balancer a8bfa26552e8511e7bee4324285f6a4a for service default/myservice with UID 8bfa2655-2e85-11e7-bee4-324285f6a4af: Requested cloud provider IP 1.1.1.1 is not available. The following cloud provider IPs are available: <list_of_IP_addresses>
    ```
    {: screen}

<br />




## 釋放已使用的 IP 位址
{: #free}

您可以刪除使用可攜式 IP 位址的負載平衡器服務，以釋放已使用的可攜式 IP 位址。

開始之前，[請為您要使用的叢集設定環境定義。](cs_cli_install.html#cs_cli_configure)

1.  列出叢集中可用的服務。

    ```
    kubectl get services
    ```
    {: pre}

2.  移除使用公用或專用 IP 位址的負載平衡器服務。

    ```
    kubectl delete service <myservice>
    ```
    {: pre}