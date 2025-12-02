---
lab:
  title: 'ラボ:Azure Arc を使用してオンプレミスの Windows サーバーを管理する'
  module: 'Module: Manage on-premises Windows servers by using Azure Arc'
---

# ラボ:Azure Arc を使用してオンプレミスの Windows サーバーを管理する
# 受講生用ラボ マニュアル

## ラボのシナリオ

組織には、独自のデータセンターでホストされているサーバーと複数の Azure サブスクリプションで構成されるハイブリッド環境があります。 組織は現在、オンプレミスおよびクラウド ベースのリソース全体で一貫した管理、メンテナンス、監視のアプローチを維持することに苦労しています。 運用モデルを合理化し、管理オーバーヘッドを最小限に抑えるために、Azure Arc の実装を計画しています。Azure Arc を使用して、Azure Policy による組織の標準の適用、Microsoft Defender for Cloud によるサーバーの保護、Azure Monitor を使用したパフォーマンスと正常性の監視、Azure Update Manager を使用した更新プログラムの管理を行う予定です。 さらに、Azure VM 拡張機能と azcmagent CLI を利用して、すべてのサーバー間で一貫した構成を確保する予定です。

## 目標

このラボを完了すると、次のことができるようになります。

- [オンプレミスの Windows サーバーを Azure Arc にオンボードする](https://github.com/MicrosoftLearning/Deploy-and-manage-Azure-Arc-enabled-Servers/blob/master/Instructions/Labs/LAB_01_manage_windows_servers_using_azure_arc.md#exercise-1-onboard-windows-servers-to-azure-arc) 
- [Azure Arc と Azure Policy を使用してオンプレミスの Windows サーバーを管理する](https://github.com/MicrosoftLearning/Deploy-and-manage-Azure-Arc-enabled-Servers/blob/master/Instructions/Labs/LAB_01_manage_windows_servers_using_azure_arc.md#exercise-2-manage-azure-arc-enabled-windows-servers-by-using-azure-policy)
- [Microsoft Defender for Cloud を使用してオンプレミスの Windows サーバーを保護する](https://github.com/MicrosoftLearning/Deploy-and-manage-Azure-Arc-enabled-Servers/blob/master/Instructions/Labs/LAB_01_manage_windows_servers_using_azure_arc.md#exercise-3-enhance-security-of-azure-arc-enabled-windows-servers-by-using-microsoft-defender-for-cloud)
- [Azure Monitor を使用してオンプレミスの Windows サーバーを監視する](https://github.com/MicrosoftLearning/Deploy-and-manage-Azure-Arc-enabled-Servers/blob/master/Instructions/Labs/LAB_01_manage_windows_servers_using_azure_arc.md#exercise-4-monitor-azure-arc-enabled-windows-servers-by-using-azure-monitor)
- [Azure Update Manager を使用してオンプレミスの Windows サーバーを更新する](https://github.com/MicrosoftLearning/Deploy-and-manage-Azure-Arc-enabled-Servers/blob/master/Instructions/Labs/LAB_01_manage_windows_servers_using_azure_arc.md#exercise-5-manage-updates-of-azure-arc-enabled-windows-servers-by-using-azure-update-manager)
- [Azure VM 拡張機能と azcmagent CLI を使用してオンプレミスの Windows サーバーを構成する](https://github.com/MicrosoftLearning/Deploy-and-manage-Azure-Arc-enabled-Servers/blob/master/Instructions/Labs/LAB_01_manage_windows_servers_using_azure_arc.md#exercise-6-configure-on-premises-windows-servers-by-using-azure-vm-extensions-and-azcmagent-cli)

## ラボの所要時間

合計推定時間:230 分

## 要件

- このラボで使用する Azure リージョンに、2 つの Azure 仮想マシン (VM) のデプロイに対応するのに十分な数の vCPU を使用できる Microsoft Azure サブスクリプション。
- このラボで使用する Azure サブスクリプション内の所有者ロールを持つ職場または学校アカウント、または Microsoft アカウント。 また、このアカウントは、そのサブスクリプションに関連付けられている Microsoft Entra テナントのグローバル管理者であることも必要です。 
- Azure Cloud Shell と互換性のある Web ブラウザーを備え、Microsoft クラウドにアクセスできるラボ コンピューター。

## 演習

このラボは、次の演習で構成されます。

- Windows サーバーを Azure Arc にオンボードする
- Azure Policy を使用して Azure Arc 対応 Windows サーバーを管理する
- Microsoft Defender for Cloud を使用して Azure Arc 対応 Windows サーバーのセキュリティを強化する
- Azure Monitor を使用して Azure Arc 対応 Windows サーバーを監視する
- Azure Update Manager を使用して Azure Arc 対応 Windows サーバーの更新プログラムを管理する

### 演習 1:Windows サーバーを Azure Arc にオンボードする

予測される所要時間: 60 分

この演習では、次のことを行います。

- Azure Resource Manager テンプレートを使用して Azure リソースをデプロイする
- Azure VM を Azure Arc に接続するためのオペレーティング システムの前提条件を実装する
- オンプレミス サーバーを Azure Arc に接続するための準備を行う
- Windows Admin Center を使用して Windows Server を Azure Arc に接続する 
- Windows サーバーを非対話形式で大規模に Azure Arc に接続する

> **注:**  Windows Server リソースを Azure Arc に接続するには、ここに提示する方法以外にもいくつかの方法があります。 包括的な一覧については、「[Azure Connected Machine エージェントのデプロイ オプション](https://learn.microsoft.com/en-us/azure/azure-arc/servers/deployment-options)」を参照してください。

> **注:**  実用的な理由から、すべてのラボで、Azure VM を使用してオンプレミスの Windows サーバーをエミュレートします。 一般に、Azure VM には既に同じ機能 (Azure Resource Manager、VM 拡張機能、マネージド ID、Azure Policy のサポートなど) がネイティブに含まれているため、Azure Arc には接続されません。 実際のところ、Azure VM に Azure Arc をインストールしようとすると、このような構成はサポートされていないことを示すエラー メッセージが表示されます。 ただし、評価とテストの目的で Azure VM に Azure Arc をインストールすることはできます。 このトピックの詳細については、Microsoft Learn の記事「[Azure 仮想マシンで Azure Arc 対応サーバーを評価する](https://learn.microsoft.com/en-us/azure/azure-arc/servers/plan-evaluate-on-azure-virtual-machine)」を参照してください。

> **注:**  ここでは Windows Server に焦点を当てていますが、Azure Arc の範囲は Linux サーバー、Kubernetes クラスター、Azure データ サービス、SQL Server、Azure Stack HCI や VMware vSphere などの仮想化プラットフォームにまで及ぶことに注意してください。

#### タスク 1:Azure Resource Manager テンプレートを使用して Azure リソースをデプロイする

このタスクでは、オンプレミス環境で Windows サーバーをエミュレートするために使用される Azure コンピューティング リソースをデプロイします。 このデプロイでは、Windows Server 2022 を実行する **arclab-winvm0** と **arclab-winvm1** という名前の 2 つの Azure VM がプロビジョニングされます。

1. ラボ コンピューターから Web ブラウザーを起動し、[https://portal.azure.com](https://portal.azure.com) にある Azure portal に移動します。
1. メッセージが表示されたら、このラボで使用する Azure サブスクリプション内の所有者ロールを持つ職場または学校アカウント、または Microsoft アカウントでサインインします。
1. Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

    > **注**:メッセージが表示されたら、**[PowerShell]** を選択し、**[作業の開始]** ペインで **[ストレージ アカウントは不要です]** を選択し、**[サブスクリプション]** ドロップダウン リストで、このラボで使用している Azure サブスクリプションを選択し、**[適用]** を選択します。 

1. Cloud Shell ペインの PowerShell セッションで、次のコマンドを実行して、デプロイに使用する Azure Resource Manager (ARM) テンプレートをホストするリポジトリのシャロー クローンを作成し、現在のディレクトリをそのテンプレートとそのパラメーター ファイルの場所に設定します。

    ```
    Set-Location -Path $HOME
    Remove-Item -Path ./Deploy-and-manage-Azure-Arc-enabled-Servers -Recurse -Force
    git clone --depth 1 https://github.com/MicrosoftLearning/Deploy-and-manage-Azure-Arc-enabled-Servers.git
    Set-Location -Path ./Deploy-and-manage-Azure-Arc-enabled-Servers/Allfiles/Templates
    ```

1. Cloud Shell ペインで、次のコマンドを実行して、ターゲット リソース グループ名を指定する変数 `$rgName` を `arclab-infra-RG` に設定します。

    ```powershell
    $rgName = 'arclab-infra-RG'
    ```

1. 次のコマンドを実行して、変数 `$location` の値を、ラボ リソースをデプロイする Azure リージョンの名前に設定します (プレースホルダー `<Azure_region>` をそのリージョンの名前に置き換えます)。

    ```powershell
    $location = '<Azure_region>'
    ```

    > **注**:注:Azure PowerShell コマンドレット `(Get-AzLocation).Location` を実行すると、使用可能な Azure リージョンの名前を一覧表示できます。

1. 次のコマンドを実行して、選択した Azure リージョンに **arclab-infra-RG** という名前のリソース グループを作成します。

    ```powershell
    New-AzResourceGroup -Name $rgName -Location $location -Force
    ```

1. 次のコマンドを実行して、変数 `$deploymentName`の一意の値を設定します。

    ```powershell
    $deploymentName = 'arclabinfra' + $(Get-Date -Format 'yyyy-MM-dd-hh-mm')
    ```

1. 次のコマンドを実行して、ローカル管理者のパスワードの名前を設定します (プレースホルダー `<Password>` を、デプロイに含まれる Azure VM にサインインするために使用する任意のパスワードに置き換えます)。

    > **注**:パスワードは必ず 12 文字以上で、小文字、大文字、1 つ以上の数字、少なくとも 1 つの特殊文字の組み合わせを含めるようにしてください

    ```powershell
    $adminPassword = ConvertTo-SecureString '<Password>' -AsPlainText -Force
    ```

1. 次のコマンドを実行して、リモート デスクトップ経由で Azure VM に接続するかどうかに応じて、変数 `$deployBastion` の値を数値 1 (`true`) または 0 (`false`) に設定します。

    > **注**:Azure Bastion を使用することをお勧めします。これにより、インターフェイスに割り当てられたパブリック IP アドレスを介して Azure VM が直接公開されなくなり、セキュリティが強化されます。 ただし、Azure Bastion を使用すると追加の[コスト](https://azure.microsoft.com/en-us/pricing/details/azure-bastion/)が発生し、デプロイの時間が長くなることに注意してください。 

    ```powershell
    [bool] $deployBastion = <1/0>
    ```

1. 次のコマンドを実行してデプロイを開始します。

    > **注**:プレースホルダー `<Password>` を、デプロイに含まれる Azure VM にサインインするために使用するパスワードに置き換えます。 パスワードは必ず 12 文字以上で、小文字、大文字、1 つ以上の数字、少なくとも 1 つの特殊文字の組み合わせを含めるようにしてください。 Azure Bastion を使用して Azure VM に接続する (リモート デスクトップ経由) かどうかに応じて、`deployBastion` パラメーターの値を `true` または `false` に設定します。

    ```powershell
    New-AzResourceGroupDeployment -Name $deploymentName `
                                  -ResourceGroupName $rgName `
                                  -TemplateFile ./azuredeploylab.json `
                                  -TemplateParameterFile ./azuredeploy.parameters.json `
                                  -adminPassword $adminPassword -deployBastion $deployBastion
    ```

    > **注**:デプロイには、Azure Bastion を含めない場合は約 1 分、含める場合は約 10 分かかります。 デプロイが完了するまで待ってから、次のタスクに進みます。

#### タスク 2:Azure VM を Azure Arc に接続するためのオペレーティング システムの前提条件を実装する

このタスクでは、Azure VM を Azure Arc にオンボードするためのオペレーティング システムの前提条件が満たされるように、それらの条件を実装します。

> **注**:これらの前提条件は、Azure VM を使用してオンプレミス サーバーをシミュレートするこの特定のラボ シナリオにのみ適用されることに留意することが重要です。 Azure 以外の VM のオンボードを伴う運用シナリオで、このような前提条件を実装する必要はありません。

> **注**:Azure Arc で使用される Connected Machine Agent のインストールを Azure VM でサポートするには、4 つのタスクを実装する必要があります。 1 つ目のタスクでは、値が **true** で、**MSFT_ARC_TEST** という名前のシステム環境変数を作成します。 2 つ目のタスクでは、すべての Azure VM 拡張機能を削除します。 3 つ目のタスクでは、Azure VM ゲスト エージェント サービスを停止して無効にする必要があります。 最後に、セキュリティが強化された Windows Defender ファイアウォールを使用して実現できる Azure Instance Metadata Service (IMDS) エンドポイントへのアクセスをブロックする必要もあります。 

1. ラボ コンピューターから、Azure portal が表示されている Web ブラウザー ウィンドウのページの上部にある **[検索]** テキスト ボックスで、**[仮想マシン]** を検索して選択します。
1. **[仮想マシン]** ページで、**[arclab-winvm0]** エントリを選択します。 
1. **[arclab-winvm0]** ページの左側にある垂直メニューの **[設定]** セクションで、**[拡張機能とアプリケーション]** を選択します。 
1. **[拡張機能]** タブに拡張機能が表示されている場合は、それらの拡張機能をそれぞれ選択し、拡張機能の詳細ペインで **[アンインストール]** を選択します。

    > **注**:1 つの拡張機能のアンインストールが完了するまで待ってから、次の拡張機能に進んでください。

1. **[arclab-winvm0]** ページで、**[接続]** メニュー オプションを使用して Windows Server に接続します。

    > **注**:前のタスクのデプロイに Azure Bastion を含めたかどうかに応じて、**[接続]** オプションまたは **[Bastion 経由で接続]** オプションを使用します。 

    > **注**:Azure Bastion を使用しない場合は、**[RDP ファイルのダウンロード]** オプションを使用して RDP ファイルをダウンロードして開きます。

    > **注**:Azure Bastion を使用する場合は、Microsoft Edge ポップアップ ブロッカーの潜在的な影響を排除するために、**[新しいブラウザー タブで開く]** オプションを選択解除します。

1. 認証を求められたら、ユーザー名 **arclabadmin** と、デプロイ時に指定したパスワードを入力します。
1. **arclab-winvm0** へのリモート デスクトップ セッション内で、スタート メニューから **[Windows PowerShell ISE]** を起動します。 
1. **[管理者: Windows PowerShell ISE]** ウィンドウで、次のコマンドを実行してシステム環境変数 **MSFT_ARC_TEST** を作成し、その値を **true** に設定します

    ```powershell
    [System.Environment]::SetEnvironmentVariable("MSFT_ARC_TEST",'true', [System.EnvironmentVariableTarget]::Machine)
    ```

1. **[管理者: Windows PowerShell ISE** ウィンドウで、次のコマンドを実行して、Azure VM ゲスト エージェントの状態を停止して無効にします。

    ```powershell
    Set-Service WindowsAzureGuestAgent -StartupType Disabled -Verbose
    Stop-Service WindowsAzureGuestAgent -Force -Verbose
    ```

1. **[管理者: Windows PowerShell ISE** ウィンドウで、次のコマンドを実行して、セキュリティが強化された Windows Defender ファイアウォールを作成し、Azure IMDS エンドポイントへの送信アクセスをブロックします。

    ```powershell
    New-NetFirewallRule -Name BlockAzureIMDS -DisplayName "Block access to Azure IMDS" -Enabled True -Profile Any -Direction Outbound -Action Block -RemoteAddress 169.254.169.254 
    ```

> **注**:このタスクのすべての手順を繰り返して、Azure Arc の前提条件に接続し、**arclab-winvm1** に実装します。

#### タスク 3:オンプレミス サーバーを Azure Arc に接続するための準備を行う

このタスクでは、オンプレミスのリソースを Azure Arc に接続するためのラボ環境を準備します。このためには、ターゲットの Azure サブスクリプションに次のリソース プロバイダーを登録する必要があります。

- Microsoft.HybridCompute
- Microsoft.GuestConfiguration
- Microsoft.HybridConnectivity
- Microsoft.Compute.

1. 必要に応じて、**arclab-winvm0** へのリモート デスクトップ セッションに切り替えます。
1. **arclab-winvm0** へのリモート デスクトップ セッション内で、Web ブラウザーを起動し、Azure portal ([https://portal.azure.com](https://portal.azure.com)) に移動します。
1. メッセージが表示されたら、このラボで使用している Azure サブスクリプションの所有者または共同作成者のロールを持つ職場または学校アカウント、または Microsoft アカウントでサインインします。
1. Azure portal 上の Cloud Shell で PowerShell セッションを開始します。 

    > **注**:メッセージが表示されたら、**[PowerShell]** を選択し、**[作業の開始]** ペインで **[ストレージ アカウントは不要です]** を選択し、**[サブスクリプション]** ドロップダウン リストで、このラボで使用している Azure サブスクリプションを選択し、**[適用]** を選択します。 

1. Cloud Shell ペイン内の PowerShell セッションで、次のコマンドを実行して、Azure Arc 対応サーバーの実装に必要な Azure リソース プロバイダーを登録します。

    ```powershell
    Register-AzResourceProvider -ProviderNamespace Microsoft.HybridCompute
    Register-AzResourceProvider -ProviderNamespace Microsoft.GuestConfiguration
    Register-AzResourceProvider -ProviderNamespace Microsoft.HybridConnectivity
    Register-AzResourceProvider -ProviderNamespace Microsoft.AzureArcData
    ```

    > **注:**  登録が完了するまで待たずに、次の手順に進んでください。 登録は、数分以内に完了します。 

#### タスク 4:Windows Admin Center を使用して Windows Server を Azure Arc に接続する

このタスクでは、Windows Admin Center をインストールして Azure に登録し、それを使用して **arclab-winvm0** を Azure Arc に接続します。

1. **arclab-winvm0** へのリモート デスクトップ セッション内で、**[管理者: Windows PowerShell ISE]** ウィンドウに切り替え、スクリプト ペインを開きます。
1. Windows PowerShell ISE スクリプト ペインで、Windows Admin Center をインストールする次のスクリプトを入力し、ツール バーの緑色の矢印アイコンを選択して実行します。

    ```powershell
    Invoke-WebRequest 'https://aka.ms/WACDownload' -OutFile "$pwd\WAC.msi"
    $msiArgs = @("/i", "$pwd\WAC.msi", "/qn", "/L*v", "log.txt", "SME_PORT=443", "SSL_CERTIFICATE_OPTION=generate")
    Start-Process msiexec.exe -Wait -ArgumentList $msiArgs
    ```

    > **注:**  Windows Admin Center のインストールが完了するまで待ちます。 インストールには約 3 分かかります。 [https://localhost](https://localhost) 経由でアクセス可能な Windows Admin Center ゲートウェイ コンポーネントがプロビジョニングされ、60 日間有効な自己署名証明書によってセキュリティで保護されます。

    > **注:**  次に、Windows Admin Center を Azure に登録する必要があります。

1. **arclab-winvm0** へのリモート デスクトップ セッション内で、Web ブラウザーを起動し、[https://localhost](https://localhost) ページに移動します。

    > **注:**  実際のサーバー名ではなく、必ず **localhost** 名を使用してください。

1. 「**接続がプライベートではありません**」という警告が表示されたら、**[詳細]**、**[localhost (unsafe) を続行]** の順に選択します。

    > **注:**  この警告は、ターゲット サイトで自己署名証明書が使用されていることが原因で表示されたと考えられます。

1. 認証を求められたら、**arclabadmin** としてサインインします。
1. インストールが成功したことを確認するペインを閉じ、Windows Admin Center 拡張機能の更新が完了するまで待ってから、その完了を確認します。
1. Windows Admin Center の **[すべての接続]** ページで、ページの右上隅にある歯車アイコンを選択します。
1. **[設定 \| アカウント]** ページの **[Azure アカウント]** セクションで **[Azure に登録する]** を選択し、**[Azure に登録する]** ペインで **[登録]** を選択します。
1. **[Windows Admin Center で Azure の使用を開始する]** ペインの登録プロセスの手順 2 で、**[コピー]** を選択して、登録コードをクリップボードにコピーします。
1. 登録プロセスの手順 3 で、「**コードを入力してください**」というテキストの横にあるリンクを選択します。
 
    > **注:**  これにより、Microsoft Edge ウィンドウで別のタブが開き、[コードの入力] ページが表示されます。

1. **[コードの入力]** ペインに、クリップボードにコピーしたコードを貼り付け、**[次へ]** を選択します。
1. サインインを求められたら、先ほど Azure サブスクリプションにアクセスするために使用したものと同じ資格情報を使用します。
1. 「**Windows Admin Center にサインインしようとしていますか?**」という質問に対する確認を求められたら、**[続行]** を選択します。
1. サインインが成功したことを確認し、Web ブラウザー ウィンドウの新しく開いたタブを閉じます。
1. **[Windows Admin Center で Azure の使用を開始する]** ペインに戻り、登録プロセスの手順 4 で、**[新規作成]**、**[接続]** の順に選択します。

    > **注:**  これにより、このラボで使用している Azure サブスクリプションに関連付けられている Microsoft Entra テナントにサービス プリンシパルが生成されます。

1. **[Windows Admin Center で Azure の使用を開始する]** ペインの登録プロセスの手順 5 で、**[サインイン]** を選択します。
1. サインインを求められたら、ラボで Azure サブスクリプションへのアクセスを認証するためにこれまで使用していたものと同じ Microsoft Entra ID 資格情報を入力します。
1. メッセージが表示されたら、**[要求されるアクセス許可]** ポップアップ ウィンドウで、アプリケーションに必要なアクセス許可を確認し、**[組織に代わって同意する]** チェックボックスをオンにし、**[承諾]** を選択します。
1. Windows Admin Center ページの **[Azure に登録する]** ペインで、登録が成功したことを確認します。

    > **注:**  次に、Windows Admin Center を使用して Windows Server を Azure Arc に接続します。

1. Windows Admin Center が表示されている Web ブラウザー ウィンドウで、**[設定]** (青いヘッダー) を選択し、ドロップダウン リストで **[すべての接続]** を選択し、接続の一覧で [arclab-winvm0 (ゲートウェイ)] エントリを選択します。
1. Windows Admin Center インターフェイスの **[arclab-winvm0]** ページの左側にあるナビゲーション メニューで、**[Azure ハイブリッド センター]** を選択します。
1. **[Azure ハイブリッド センター]** ペインの **[利用可能なサービス]** タブの **[Azure Arc のセットアップ]** セクションで、**[セットアップ]** を選択します。
1. **[Azure Arc のセットアップ]** ペインで、次の操作を実行します。

    1. **[サブスクリプション]** ドロップダウン リストに、このラボで使用している Azure サブスクリプションの名前が表示されていることを確認します。
    1. **[リソース グループ]** セクションで、**[新規作成]** オプションが選択されていることを確認し、「**arclab-servers-RG**」と入力します。
    1. **[Azure リージョン]** ドロップダウン リストで、このラボで先ほど Azure VM をデプロイするために選択したものと同じ Azure リージョンの名前を選択します。 

    > **注:**  この Azure リージョンには、Azure Arc 対応サーバーのメタデータが格納されます。

1. **[セットアップ]** を選択して、**arclab-winvm0** を Azure Arc 対応サーバーとして構成する手順に進みます。

    > **注:**  サーバーは Azure に接続され、Connected Machine エージェントをダウンロードしてインストールし、Azure Arc に登録します。進行状況を追跡するには、Windows Admin Center ツール バーの右上隅にある **[通知]** (ベルの形) アイコンを選択します。 インストールによってコマンド プロンプト ウィンドウが表示され、Connected Machine エージェントの **azcmagent.exe** インストーラーの実行コンテキストが提供されます。 インストールが完了するまで待ちます。 これには 2 分ほどかかる場合があります。 

1. 成功した結果を検証するには、Azure portal が表示されている Web ブラウザー ウィンドウに切り替え、**[検索]** テキスト ボックスで **[Azure Arc]** を検索して選択します。
1. **[Azure Arc]** ページの左側にあるナビゲーション メニューの **[Azure Arc リソース]** セクションで、**[マシン]** を選択します。
1. [Azure Arc \| マシン] ページで、Arc エージェントの状態が **[接続済み]** である Azure Arc 対応マシンの一覧に、**[arclab-winvm0]** エントリが表示されていることを確認します。

#### タスク 5:Windows サーバーを非対話形式で大規模に Azure Arc に接続する

このタスクでは、サービス プリンシパルを使用して Windows Server を Azure Arc に接続します。これは、大規模なシナリオで Azure Arc へのサーバーの接続を容易にする非対話型の方法の 1 つを示しています。 このサービス プリンシパルを (前のタスクで使用した) ID の代わりに使用して、ターゲットの Azure サブスクリプションへのアクセスを承認することができます。 この方法を実装するには、このラボで先ほどデプロイした **arclab-winvm1** という名前の 2 つ目の Azure VM を使用します。

> **注:**   Azure portal から使用できるテンプレート スクリプトを使用して、Azure Arc に接続するプロセスを自動化することができます。 最初にスクリプトを生成します。

1. 必要に応じて、**arclab-winvm1** へのリモート デスクトップ セッションに切り替えます。
1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Web ブラウザーを起動し、Azure portal ([https://portal.azure.com](https://portal.azure.com)) に移動します。
1. メッセージが表示されたら、このラボで使用している Azure サブスクリプション内の所有者ロールを持つ職場または学校アカウント、または Microsoft アカウントでサインインします。
1. Azure portal の **[検索]** テキスト ボックスで、**[Azure Arc]** を検索して選択します。
1. **[Azure Arc]** ページの左側にあるナビゲーション メニューの **[Azure Arc リソース]** セクションで、**[マシン]** を選択します。
1. **[Azure Arc \| マシン]** ページで **[追加/作成]** を選択し、ドロップダウン メニューから **[マシンの追加]** を選択します。
1. **[Azure Arc でサーバーを追加]** ページの **[複数のサーバーの追加]** タイルで、**[スクリプトの生成]** を選択します。
1. **[Azure Arc で複数のサーバーを追加]** ページの **[基本情報]** タブで、次の設定を指定します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**arclab-servers-RG**|
    |リージョン|前のタスクで選択したものと同じ Azure リージョンの名前|
    |オペレーティング システム|**Windows**|
    |接続方法|**パブリック エンドポイント**|

1. **[認証]** セクションの **[サービス プリンシパル]** ドロップダウン リストで、**[新規作成]** を選択します。 
1. **[新しい Azure Arc サービス プリンシパル]** ペインで、次の設定を指定し、**[作成]** を選択します。

    |設定|値|
    |---|---|
    |名前|**arclab-servers-sp**|
    |スコープの割り当てレベル|**リソース グループ**|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**arclab-servers-RG**|
    |クライアント シークレットの説明|**arclab-servers-sp-secret**|
    |有効期限|**1 週間**|
    |ロールの割り当てロール|**Azure Connected Machine のオンボード**|

    > **注:**  設定に基づいて有効期限を調整します。

    > **注:**  Azure Connected Machine のオンボード ロールは、大規模なオンボードに使用できます。 Azure では、新しい Azure Arc 対応サーバーの読み取りまたは作成のみが可能です。 既に登録済みのサーバーの削除や、拡張機能の管理には使用できません。 ベスト プラクティスとして、マシンの大規模なオンボードに使用される Microsoft Entra サービス プリンシパルにのみこのロールを割り当てることを検討する必要があります。 Azure Connected Machine のリソース管理者ロールは、マシンの読み取り、変更、再オンボード、削除のアクセス許可を付与します。 このロールは、Azure Arc 対応サーバーの管理をサポートするように設計されていますが、リソース グループまたはサブスクリプション内の他のリソースの管理はサポートしていません。

1. **[サービス プリンシパル ID とシークレットのダウンロード]** ダイアログ ボックスが表示されたら、**[ダウンロードして閉じる]** を選択します。

    > **注:**  ダウンロードを開始する前に、**[Azure Arc で複数のサーバーを追加]** セクションで、新しく作成したサービス プリンシパルがオンになっていることを確認します。

    > **重要:** このダイアログ ボックスを閉じると、クライアント シークレットを再度取得することはできません。

1. ブラウザー ウィンドウの通知領域で **[ファイルを開く]** を選択して、新しく作成したサービス プリンシパルの ID とシークレットの値を含む **servicePrincipal.txt** ファイルを開きます。
1. **[Azure Arc で複数のサーバーを追加]** ページの **[基本情報]** タブに戻り、**[スクリプトのダウンロードと実行]** を選択します。
1. **[Azure Arc で複数のサーバーを追加]** の **[スクリプトのダウンロードと実行]** タブで、**[デプロイ方法]** オプションが **[基本]** に設定されていることを確認し、**[2. スクリプトをダウンロードし、サービス プリンシパルの資格情報を追加する]** というラベルの付いたセクションで、**[ダウンロード]** を選択します。
1. メッセージが表示されたら、Web ブラウザー ウィンドウの右上隅にある **[保持]** を選択してダウンロードを続行します。

    > **注:**  Windows の場合、スクリプトの名前は **OnboardingScript.ps1** です。

    > **注:**  このスクリプトでは、**azcmagent** コマンドを使用し、作成したサービス プリンシパルを参照して、Azure Arc への接続を確立します。 コマンドを正常に実行するには、スクリプトのそれぞれ 29 行目と 30 行目にあるプレースホルダー `<ServicePrincipalId>` と `<ENTER SECRET HERE>` を、ダウンロードした **servicePrincipal.txt** ファイル内にあるサービス プリンシパル ID とシークレットの値に置き換える必要があります。

1. **[管理者: Windows PowerShell ISE]** ウィンドウで、ダウンロードした **OnboardingScript.ps1** ファイルを開きます。
1. **[管理者: Windows PowerShell ISE]** ウィンドウのスクリプト ペインで、スクリプトのそれぞれ 29 行目と 30 行目にあるプレースホルダー `<ServicePrincipalId>` と `<ENTER SECRET HERE>` を、ダウンロードした **servicePrincipal.txt** ファイルに含まれているサービス プリンシパル ID とシークレットの値に置き換えます。
1. **[管理者: Windows PowerShell ISE]** で、**OnboardingScript.ps1** ファイルへの変更を保存します。
1. **[管理者: Windows PowerShell ISE]** ウィンドウで次のコマンドを実行して、RemoteSigned PowerShell 実行ポリシーが適用されている環境で、ダウンロードしたスクリプトを実行できるようにします。

    ```powershell
    Unblock-File -Path $env:USERPROFILE\DOWNLOADS\OnboardingScript.ps1
    ```

1. **[管理者: Windows PowerShell ISE]** ウィンドウのスクリプト ペインで別のタブを開き、それを使用して次のスクリプトを実行して、ローカル サーバーでスクリプトを実行します。その結果として、ローカル サーバーが Azure Arc に接続されます。

    ```powershell
    $scriptName = 'OnboardingScript.ps1'
    $scriptPath = "$env:USERPROFILE\Downloads\$scriptName"
    $remoteDirectoryName = 'Temp'
    $servers = 'arclab-winvm1'
 
    $servers | ForEach-Object {New-Item -ItemType Directory -Path (Join-Path "\\$_\c`$\" "$remoteDirectoryName") -Force}
    $servers | ForEach-Object {Copy-Item -Path $scriptPath -Destination (Join-Path "\\$_\c`$\" "$remoteDirectoryName")}
 
    Invoke-Command -ComputerName $servers -ScriptBlock {
       param($directory, $file)
       & $("c:\$directory\$file")
    } -ArgumentList ($remoteDirectoryName, $scriptName)
    ```

    > **注:**  このスクリプトは、ターゲット サーバーに C:\Temp ディレクトリを作成し、それをスクリプトにコピーし、PowerShell リモート処理を使用してスクリプトの実行を開始します。

    > **注:**  明らかに、この場合は、サーバー上でスクリプトをローカルで実行するだけで済みます。 使用した方法は、`$servers` 変数の値にサーバーの名前を追加するだけで、任意の数のサーバーに適用できる方法を示すことを目的としています。

    > **注:** スクリプトが完了するのを待ちます。 これには 3 分ほどかかる場合があります。

1. 前のタスクと同様に、成功した結果を検証するには、Azure portal が表示されている Web ブラウザー ウィンドウで、**[Azure Arc \| マシン]** ページに移動し、Arc エージェントの状態が **[接続済み]** である Azure Arc 対応マシンの一覧に、**[arclab-winvm1]** エントリが表示されていることを確認します。

### 演習 2:Azure Policy を使用して Azure Arc 対応 Windows サーバーを管理する

推定所要時間: 25 分

この演習では、次のことを行います。

- Azure Arc 対応 Windows サーバーを対象とするポリシー割り当てを作成する
- ポリシー割り当ての結果を評価する

> **注:**  Azure Policy では、Arc 対応 Windows Severs のコンプライアンス評価を扱うさまざまなシナリオがサポートされています。 このようなシナリオの例については、「[チュートリアル: 準拠していないリソースを識別するためのポリシー割り当てを作成する](https://learn.microsoft.com/en-us/azure/azure-arc/servers/learn/tutorial-assign-policy-portal)」を参照してください。

#### タスク 1:Azure Arc 対応 Windows サーバーを対象とするポリシー割り当てを作成する

このタスクでは、このラボの前の演習でデプロイした Arc 対応 Windows サーバーを含むリソース グループに組み込みのポリシーを割り当てます。 このポリシーにより、Azure Monitor エージェントが自動的にインストールされ、このラボの後半で Arc 対応マシンの監視機能の実装が容易になります。

1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウにある **[検索]** テキスト ボックスで、**[ポリシー]** を検索して選択します。
1. **[ポリシー]** ページの左側にある垂直ナビゲーション メニューの **[オーサリング]** セクションで、**[定義]** を選択します。
1. **[ポリシー \| 定義]** ページの [検索] ボックスに「**Windows Arc 対応マシンの構成**」と入力し、結果の一覧で **[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** を選択します。
1. **[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** ポリシー定義ページで、ポリシー定義と使用可能な効果を確認し、**[ポリシーの割り当て**] を選択します。
1. **[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** ポリシー割り当てページの **[基本情報]** タブで、**[スコープ]** テキスト ボックスの右側にある省略記号ボタンを選択し、**[スコープ]** ペインの **[サブスクリプション]** ドロップダウン リストで、このラボで使用している Azure サブスクリプションを選択し、**[リソース グループ]** ドロップダウン リストで **[arclab-servers-RG]** を選択した後、**[選択]** ボタンを選択します。
1. **[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** ポリシーの割り当てページの **[基本情報]** タブに戻ると、**[ポリシーの適用]** が既定で有効になっていることを確認し、**[次へ]** を選択します。
1. このポリシー定義には、入力または確認を必要とするパラメーターが含まれていないため、**[パラメーター]** タブでは **[次へ]** を選択します。
1. **[修復]** タブで **[修復タスクの作成]** チェック ボックスをオンにし、**[システム割り当てマネージド ID]** オプションがオンになっていることを確認し、**[次へ]** を選択します。

    > **注:**  これは、既存のリソースに割り当てを適用するために必要です。 通常、ポリシー設定は、割り当ての作成後に作成されたリソースに適用されます。 ただし、修復タスクが構成されている場合、**deployIfNotExist** および **Modify** 効果を含むポリシーは、既存のリソースに適用されます。

1. **[非準拠メッセージ]** タブで、**[次へ]** を選択します。
1. **[確認および作成]** タブで、**[作成]** を選択します。

#### タスク 2:ポリシーの結果を評価する

このタスクでは、ポリシー割り当ての結果を確認します。

1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウで、**[ポリシー \| 定義]** ページに戻り、左側の垂直メニューで **[コンプライアンス]** を選択します。
1. **[ポリシー \| コンプライアンス]** ページのポリシー割り当ての一覧で、**[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** エントリを見つけます。 おそらく、この時点では、ポリシー割り当ての対象となる両方のリソースの **[コンプライアンス]** の状態は、ポリシーの一覧で [未開始] または **[準拠していない]** と表示されます。**

    > **注:**  その場合は、コンプライアンスの状態が **[準拠していない]** に変わるまで待ちます。 これには約 3 分かかる場合があります。 新しく作成されたポリシー割り当てとその更新された状態を表示するには、ページの更新が必要になる場合があります。

1. **[ポリシー \| コンプライアンス]** ページで、**[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** エントリを選択します。
1. **[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** ポリシーのコンプライアンス ページで、準拠していないリソースの詳細な一覧を確認し、ツール バーの **[修復タスクの作成]** ボタンに注意してください。

    > **注:**  この機能を使用して、修復タスクを明示的に呼び出すことができます。 ここでは、修復タスクが既に実行されているため、これは必要ありません。 

1. 修復タスクが実行されているかどうかを確認するには、**[ポリシー \| コンプライアンス]** ページに戻り、**[修復]** を選択し、**[ポリシー \| 修復]** ページで **[修復タスク]** タブを選択します。
1. 修復タスクの一覧で、新しく作成されたポリシー割り当てに関連付けられている修復タスクを表す既存のエントリに注意してください。

    > **注:**  修復タスクは、**[評価中]** 状態から **[進行中]**、**[完了]** の順に移行します。 これには 15 分ほどかかる場合があります。 タスクが完了するのを待たずに、次のタスクに進みます。 後でこのタスクを見直すことを検討してください。

    > **注:**  または、**[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** ポリシーのコンプライアンス ページに移動し、ツール バーの **[アクティビティ ログ]** ボタンを選択してログ エントリを確認し、修復タスクの状態を確認することもできます。 ログを使用して、修復アクティビティの進行状況を追跡できます。 ある時点で、**deployIfNotExists** ポリシー アクションの呼び出しと Azure Monitor エージェント拡張機能のインストールを参照するエントリが表示されます。

1. 修復タスクが完了したら、**[Azure Monitor エージェントを実行するように Windows Arc 対応マシンを構成する]** ポリシーのコンプライアンス ページに戻り、4 つのリソースがすべて "準拠している" として一覧表示されていることを確認します。

    > **注:**  更新されたコンプライアンスの状態を表示するには、ページの更新が必要になる場合があります。

1. 別の方法で結果を検証するには、Azure portal で **[Azure Arc \| マシン]** ページに戻り、**[arclab-winvm1]** エントリを選択します。
1. **[arclab-winvm1]** ページの左側にある垂直ナビゲーション メニューで、**[設定]** セクションを展開し、**[拡張機能]** を選択して、インストールされている拡張機能の一覧に **[AzureMonitorWindowsAgent]** が表示されることを確認します。

### 演習 3:Microsoft Defender for Cloud を使用して Azure Arc 対応 Windows サーバーのセキュリティを強化する

予測される所要時間: 45 分

この演習では、次のことを行います。

- Azure Arc 対応 Windows サーバーの Microsoft Defender for Cloud ベースの保護を構成する 
- Azure Arc 対応 Windows サーバーの Microsoft Defender for Cloud ベースの保護を確認する 

#### タスク 1:Azure Arc 対応 Windows サーバーの Microsoft Defender for Cloud ベースの保護を構成する 

このタスクでは、Microsoft Defender for Cloud による Arc 対応 Windows サーバーの保護を有効にします。 

1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウにある **[検索]** テキスト ボックスで、**[Microsoft Defender for Cloud]** を検索して選択します。
1. **[Microsoft Defender for Cloud \| 概要]** ページの左側にあるナビゲーション メニューで、**[はじめに]** を選択します。
1. **[Microsoft Defender for Cloud \| 概要]** ページの **[アップグレード]** タブにある **[1 個のサブスクリプションで Defender for Cloud を有効にする]** セクションで、Azure サブスクリプションのエントリの横にあるチェック ボックスがオンになっていることを確認し、**[アップグレード]** を選択します。

    > **注:**  これにより、Microsoft Defender for Cloud の 30 日間の試用版が開始されます。

1. セッションが **[エージェントのインストール]** タブにリダイレクトされても、**[エージェントのインストールする]** を選択**しないでください**。

    > **注:**  Microsoft Defender for Cloud は、Azure Arc 対応サーバーの保護を含む Defender for Servers オファリング (マシン上の Defender for SQL Server を除く) について、Azure Monitor エージェントまたは Log Analytics エージェント (提供終了日は 2024 年 8 月に設定されています) に依存しません。 詳細については、「[Defender for Cloud の Azure Monitor エージェント](https://learn.microsoft.com/en-us/azure/defender-for-cloud/auto-deploy-azure-monitoring-agent)」を参照してください。

1. **[Microsoft Defender for Cloud \| はじめに]** ページの左側にあるナビゲーション メニューの **[管理]** セクションで、**[環境設定]** を選択します。
1. **[Microsoft Defender for Cloud \| 環境設定]** ページで、このラボで使用している Azure サブスクリプションを表すエントリを選択します。
1. **[設定 \| Defender プラン]** ページで、**更新されたエクスペリエンスの自動プロビジョニング**を試すかどうかを尋ねられたら、**[今すぐ試す]** を選択します。 これにより、ブラウザー セッションが **[設定と監視]** ページにリダイレクトされます。
1. **[設定と監視]** ページで、既定の設定を確認します。

    > **注:**  **[Log Analytics エージェント]** コンポーネントは、既定で無効になっています。 同様に、**[Guest Configuration エージェント (プレビュー)]** も既定で無効になっていますが、Azure Arc に接続されているサーバーの場合、このコンポーネントは既に Connected Machine エージェントに含まれています。

1. **[設定 \| Defender プラン]** ページに戻り、**[クラウド セキュリティ態勢管理 (CSPM)]** プランと **[クラウド ワークロード保護]** プランの両方を確認します。

#### タスク 2:Azure Arc 対応 Windows サーバーの Microsoft Defender for Cloud ベースの保護を確認する 

このタスクでは、Azure Arc 対応 Windows サーバーで使用できる Microsoft Defender for Cloud ベースの保護機能を確認します。

> **注:**  ラボではオンプレミスの Windows サーバーをシミュレートするために Azure VM に依存していることを考慮すると、Azure VM (オンプレミスの Windows サーバーをシミュレート) とそれに対応する Azure Arc オブジェクト (同じ Azure VM として実装) の両方が複数の Microsoft Defender for Cloud ページでリソースとして表示されるため、Azure Arc のコンテキストで Microsoft Defender for Cloud の機能を分析するのは多少困難な可能性があります。 Azure portal で表示するために使用されるさまざまなアイコンなど、いくつかの手がかりに基づいて 2 つのリソースの種類を区別できます。 これらのアイコンを識別するには、**[仮想マシン]** ページに一覧表示されているリソースと、**[Azure Arc \| マシン]** ページに一覧表示されているリソースを比較します。

1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウで、**[Microsoft Defender for Cloud \| 概要]** ページに戻ります。
1. **[Microsoft Defender for Cloud \| 概要]** ページの左側にあるナビゲーション メニューの **[全般]** セクションで、**[インベントリ]** を選択します。
1. **[Microsoft Defender for Cloud \| インベントリ]** ページでリソースの一覧を確認し、仮想マシンのエントリ **[arclab-winvm0]** および **[arclab-winvm1]** と、**[リソースの種類]** 列の値が **[マシン - Azure Arc]** である一致するエントリが含まれていることに注意してください。 

    > **注:**  **[Microsoft Defender for Cloud\| インベントリ]** ページの **[Defender for Cloud]** 列の値を確認し、Azure Arc サーバーが Microsoft Defender for Cloud にオンボードされていることを確認します。

1. **[Microsoft Defender for Cloud \| 概要]** ページの左側にあるナビゲーション メニューの **[全般]** セクションで、**[推奨事項]** を選択します。
1. **[Microsoft Defender for Cloud \| 推奨事項]** ページで、**arclab-winvm0** または **arclab-winvm1** Azure Arc 対応 Windows サーバー (Azure VM ではない) に適用できる推奨事項を探します。

    > **注:**  "**マシンで Windows Defender Exploit Guard を有効にする必要がある**" というタイトルの推奨事項を見つけることができる場合があります。

1. 2 つの Azure Arc 対応 Windows サーバーのいずれかに適用できる推奨事項が見つかった場合は、それを選択します。
1. 対応する修復ページで、修復手順を含む **[アクションの実行]** タブを確認します。 場合によっては、手順に **[修正]** というラベルの付いたボタンが含まれています。これを選択すると、**[クイック修正]** 修復ウィザードが起動されます。

    > **注:**  修復手順の適用は省略できます。 修復プロセスが完了すると、結果が Azure portal に反映されるまでに最大 24 時間かかる場合があります。

### 演習 4:Azure Monitor を使用して Azure Arc 対応 Windows サーバーを監視する

推定所要時間:30 分

この演習では、次のことを行います。

- Azure Arc 対応 Windows サーバー用に VM Insights を構成する
- Azure Arc 対応 Windows サーバーの監視機能を確認する

#### タスク 1:Azure Arc 対応 Windows サーバー用に VM Insights を構成する

このタスクでは、このラボで先ほど Azure Arc にオンボードした Windows サーバーの 1 つに対して VM Insights を構成します。 前の演習の 1 つで Azure Monitor エージェントをデプロイした方法と同様に、複数のサーバーに対して Azure Policy を使用してこの構成を管理することもできます。

> **注:**   まず、Arc 対応 Windows サーバーによって生成されたテレメトリを使用するための専用の Azure Log Analytics ワークスペースを作成します。

1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウにある **[検索]** テキスト ボックスで、**[Log Analytics ワークスペース]** を検索して選択します。
1. **[Log Analytics ワークスペース]** ページで、**[+ 作成]** を選択します。
1. **[Log Analytics ワークスペースの作成]** ページの **[基本情報]** タブで次の設定を指定し、**[確認と作成]** を選択します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**arclab-servers-RG**|
    |Name|**arclab-lamonitoringworkspace**|
    |リージョン|このラボの前の演習で使用したものと同じ Azure リージョンの名前|

1. **[レビューと作成]** タブで、検証が完了するのを待ち、**[作成]** を選択します。

    > **注**:プロビジョニング プロセスが完了するまで待ちます。 これには 1 分ほどかかります。 

1. サーバー arclab-winvm1 に署名しているときに、Azure portal が表示されている Web ブラウザー ウィンドウで、**[Azure Arc \| マシン]** ページに移動し、**[arclab-winvm1]** エントリを選択します。
1. **[arclab-winvm1]** ページの左側にある垂直ナビゲーション メニューの **[監視]** セクションで、**[Insights]** を選択します。
1. **[arclab-winvm1 \| Insights]** ページで、**[有効にする]** を選択します。 
1. **[監視の構成]** ペインの **[データ収集ルール]** ドロップダウン リストの下にある **[新規作成]** を選択します。

    > **注:**  マップ機能を有効にするために、プロセスと依存関係を含む新しいデータ収集ルールを作成します。

1. **[新しいルールの作成]** ペインで、次の設定を指定します。

    |設定|Value|
    |---|---|
    |データ収集ルール名|**arclab-datamap-DCR**|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |プロセスと依存関係を有効にする (マップ)|有効|
    |Log Analytics ワークスペース|**arclab-lamonitoringworkspace**|

1. **［作成］** を選択します
1. **[監視の構成]** ペインに戻り、**[構成]** を選択します。

    > **注:**  監視の構成が完了するのを待たずに (Azure portal ページのツール バーにあるベル アイコンを使用してアクセスできる通知領域を使用すると、進行状況を追跡できます)、次のタスクに進みます。 監視の構成プロセスが完了するまでに約 10 分かかる場合があります。

#### タスク 2:Azure Arc 対応 Windows サーバーの監視機能を確認する

このタスクでは、VM Insights によって提供される Azure Arc 対応 Windows サーバーの監視機能の設定結果を確認します。

> **注:**   監視の構成の完了には数分かかる場合があります。 その間はマップ データを表示できない可能性はありますが、パフォーマンス データにはアクセスできるはずです。 これが当てはまるかどうかを確認するには、**[有効にする]** ボタンが表示されなくなるまで、**[arclab-winvm1 \| Insights]** ページが表示されている Web ブラウザー ウィンドウを 1 分ごとに更新し、**[パフォーマンス]** タブを選択します。

> **注:**   アイドル時間を最小限に抑えるには、Azure Policy 修復タスクの実装を必要とした演習に戻り、その後でこの演習に戻ることを検討してください。

1. **[arclab-winvm1 \| Insights]** ページで、Web ブラウザー ページを更新すると、**[はじめに]**、**[パフォーマンス]**、**[マップ]** の各タブを含む更新されたインターフェイスが表示されます。
1. **[arclab-winvm1 \| Insights]** ページで **[パフォーマンス]** タブを選択し、CPU、メモリ、ディスク、ネットワークのテレメトリを表示するグラフを確認します。
1. **[arclab-winvm1 \| Insights]** ページで、**[マップ]** タブを選択します。このインターフェイスでは、監視対象サーバーで実行されているプロセスに関するデータと、その受信接続と送信接続が提供されます。 

    > **注:**  **[マップ]** 機能は、VM Insights を有効にしてから 10 から 15 分以内に使用できるようになります。 

1. 監視対象サーバーのプロセスの一覧を展開します。 プロセスの 1 つを選択して、その詳細と依存関係を確認します。
1. **[arclab-winvm1]** を選択してサーバーのプロパティを表示し、プロパティ ペインで **[ログ イベント]** を選択します。 これにより、イベントの種類とそれに対応する数をまとめたテーブルが表示されます。
1. 実際にログされたイベントを表示するには、イベントの種類のエントリのいずれかを選択します。 イベントが収集される Log Analytics ワークスペースにリダイレクトされます。 このインターフェイスから、個々のログ エントリを詳細に調べることができます。 

### 演習 5: Azure Update Manager を使用して Azure Arc 対応 Windows サーバーの更新プログラムを管理する

推定所要時間: 25 分

この演習では、次のことを行います。

- Azure Arc 対応 Windows サーバー用に Update Manager を構成する
- Azure Arc 対応 Windows サーバーの Update Manager 機能を確認する

#### タスク 1:Azure Arc 対応 Windows サーバー用に Update Manager を構成する

このタスクでは、このラボで先ほど Azure Arc にオンボードした Windows サーバーの 1 つに対して Update Manager を構成します。 

1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウで、**[Azure Arc \| マシン]** ページに移動し、**[arclab-winvm1]** エントリを選択します。
1. **[arclab-winvm1]** ページで **[更新プログラムの確認]** を選択し、**[今すぐ評価をトリガー]** で **[OK]** を選択します。

    > **注:**  評価が完了するまで待たずに、次の手順に進みます。 評価が完了するまでに約 5 分かかる場合があります。

1. **[arclab-winvm1 \| 更新プログラム]** ページで **[設定の更新]** を選択し、**[更新プログラムの設定を変更する]** ページの **arclab-winvm1** の **[定期評価]** ドロップダウン リストで **[有効]** を選択し、他の設定を確認して、**[保存]** を選択します。

    > **注:**  ホット パッチの適用は、Windows Server Datacenter Azure Edition を実行している Azure VM でのみ使用できます。 パッチ オーケストレーションは、Arc 対応サーバーには適用できません。

1. **[arclab-winvm1 \| 更新プログラム]** ページで、**[更新プログラムのスケジュール]** を選択します。 
1. **[メンテナンス構成の作成]** ページの **[基本情報]** タブで、次の設定を指定します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**arclab-servers-RG**|
    |構成名|**arclab-maintenance-configuration**|
    |リージョン|このラボの前の演習で使用したものと同じ Azure リージョンの名前|
    |メンテナンス スコープ|**ゲスト (Azure VM、Arc 対応 VM またはサーバー)**|
    |再起動の設定|**必要に応じて再起動**|

1. **[スケジュールの追加]** を選択し、**[スケジュールの追加または変更]** ペインで次の設定を指定し、**[保存]** を選択します。

    |設定|Value|
    |---|---|
    |開始時刻|ローカル タイム ゾーンの次の土曜日の **[21:00]**|
    |メンテナンス期間|**[3]** 時間と **[0]** 分|
    |繰り返し|**[毎週]**、**[日曜日]**|
    |[終了日の追加]|disabled|

1. **[メンテナンス構成の作成]** ページの **[基本情報]** タブに戻り、**[次へ: リソース>]** を選択します。
1. **[リソース]** タブで、リソースの一覧に **[arclab-winvm1]** が表示されていることを確認し、**[次へ: 動的スコープ>]** を選択します。
1. **[動的スコープ]** タブで、**[次へ: 更新>]** を選択します。

    > **注:**  動的スコープを使用すると、リソース グループ、場所、オペレーティング システムの種類、タグなどの条件を使用して、構成のスコープを絞り込むことができます。

1. **[更新プログラム]** タブで、既存の設定を確認し、変更を加えずに **[確認と作成]** を選択します。

    > **注:**  更新の分類を含めるオプションだけでなく、個々の KB ID/パッケージを含めるまたは除外するオプションも用意されています。

1. **[確認と作成]** タブで、検証が完了するまで待ってから、**[作成]** を選択します。

    > **注:**  メンテナンス構成のセットアップが完了するまで待たずに、次の手順に進みます。 セットアップは 1 分以内に完了します。 

#### タスク 2:Azure Arc 対応 Windows サーバーの Update Manager 機能を確認する

このタスクでは、Update Manager によって提供される Azure Arc 対応 Windows サーバーの更新管理機能の設定結果を確認します。

1. **[arclab-winvm1 \| 更新プログラム]** ページに戻り、**[1 回限りの更新]** を選択します。
1. **[1 回限りの更新プログラムのインストール]** ページの **[マシン]** タブで、**[arclab-winvm1]** エントリの横にあるチェックボックスをオンにし、**[次へ]** を選択します。
1. **[更新プログラム]** タブで、インストールするために選択した更新プログラムを確認し、**[次へ]** を選択します。 

    > **注:**  個々の KB ID/パッケージを含めるおよび除外するオプションが用意されています。

1. **[プロパティ]** タブの **[再起動オプション]** ドロップダウン リストで、**[再起動しない]** を選択し、**[次へ]** を選択します。 
1. **[確認とインストール]** タブで、結果の設定を確認し、**[インストール]** を選択します。

    > **注:**  更新プログラムのインストールが完了するまで待つ必要はありません。 後でインストールを確認するには、Azure portal の **[arclab-winvm1 \| 更新プログラム]** ページの **[履歴]** タブを確認するか、同じページから **arclab-winvm1** に対して別の評価をトリガーします。 

### 演習 6: Azure VM 拡張機能と azcmagent CLI を使用してオンプレミスの Windows サーバーを構成する

予測される所要時間: 45 分

この演習では、次のことを行います。

- Azure VM 拡張機能を使用して Azure Arc 対応 Windows サーバーを構成する
- azcmagent CLI を使用して Azure Arc 対応 Windows サーバーを構成する
- ラボ環境のプロビジョニングを解除する

#### タスク 1:Azure VM 拡張機能を使用して Azure Arc 対応 Windows サーバーを構成する

このタスクでは、Windows 用 Azure VM カスタム スクリプト拡張機能を使用して、このラボで先ほど Azure Arc にオンボードした Windows サーバーのいずれかにインターネット インフォメーション サービス Web サイトをインストールして構成します。 

1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウで、**[管理者: Windows PowerShell ISE]** ウィンドウを開き、必要に応じて、スクリプト ペインで新しいタブを開きます。
1. Windows PowerShell ISE スクリプト ペインで、次のスクリプトを入力します。これは、**Web-Server** サーバー ロールをインストールし、Web サーバーによってホストされている既定の Web サイトの既定のホーム ページを削除し、それを、ローカル コンピューター名を表示するカスタム ページに置き換えます。

    ```powershell
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    Remove-Item -Path 'C:\inetpub\wwwroot\iisstart.htm'
    Add-Content -Path 'C:\inetpub\wwwroot\iisstart.htm' -Value "$env:computername"
    ```

1. スクリプト ペインの内容を `C:\Temp\Install_IIS.ps1` として保存します。
1. **arclab-winvm1** へのリモート デスクトップ セッション内で、Azure portal が表示されている Web ブラウザー ウィンドウに切り替えます。 
1. Azure portal で、**[ストレージ アカウント]** を検索して選択し、**[ストレージ アカウント]** ブレードで **[+ 作成]** を選択します。
1. **[ストレージ アカウントの作成]** ページの **[基本情報]** タブで次の設定を指定し (他の設定は既定値のままにしておきます)、**[次へ]** を選択します。

    |設定|値|
    |---|---|
    |サブスクリプション|このラボで使用している Azure サブスクリプションの名前|
    |リソース グループ|**arclab-servers-RG**|
    |ストレージ アカウント名|小文字と数字で構成され、小文字から始まる 3 から 24 文字の長さのグローバルに一意の名前|
    |リージョン|このラボの前の演習で使用したものと同じ Azure リージョンの名前|
    |パフォーマンス|**Standard**|
    |冗長性|**ローカル冗長ストレージ (LRS)**|

1. **[詳細]** タブで、使用可能なオプションを確認し、既定値を受け入れて、**[次へ]** を選択します。
1. **[ネットワーク]** タブで、使用可能なオプションを確認し、既定のオプションである **[パブリック エンドポイント (すべてのネットワーク)]** を受け入れて、**[次へ]** を選択します。
1. **[データ保護]** タブで、使用可能なオプションを確認し、既定値を受け入れて、**[次へ]** を選択します。
1. **[暗号化]** タブで、使用可能なオプションを確認し、既定値を受け入れて、**[確認と作成]** を選択し、検証プロセスが完了するまで待ってから、**[作成]** を選択します。

    >**注**:Azure ストレージ アカウントが作成されるまで待ちます。 これには 2 分ほどかかります。

1. ストレージ アカウントのプロビジョニングが完了したら、**[リソースに移動]** を選択します。
1. ストレージ アカウント ページの **[データ ストレージ]** セクションで、**[コンテナー]**、**[+ コンテナー]** の順に選択します。
1. **[新しいコンテナー]** ペインの **[名前]** テキスト ボックスに、「**scripts**」と入力し、**[作成]** を選択します。
1. コンテナーの一覧が表示されているストレージ アカウント ページに戻り、**[scripts]** を選択します。
1. **[scripts]** ページで、**[アップロード]** を選択します。
1. **[BLOB のアップロード]** ペインで **[ファイルの参照]** を選択し、**[開く]** ダイアログ ボックスで `C:\Temp` フォルダーに移動し、このタスクで先ほど作成した `Install_IIS.ps1` ファイルを選択して **[開く]** を選択し、**[BLOB のアップロード]** ページに戻り、**[アップロード]** を選択します。
1. Azure portal で、**[Azure Arc \| マシン]** ページに移動し、**[arclab-winvm1]** エントリを選択します。
1. **[arclab-winvm1]** ページの左側にある垂直ナビゲーション メニューの **[設定]** セクションで、**[拡張機能]** を選択します。
1. **[arclab-winvm1 \| 拡張機能]** ページで、**[+ 追加]** を選択します。
1. **[拡張機能のインストール]** ページで、**[Windows 用カスタム スクリプト拡張機能 - Azure Arc]**、**[次へ]** の順に選択します。
1. **[Windows 用カスタム スクリプト拡張機能の構成 - Azure Arc 拡張機能]** ページで、**[スクリプト ファイル (必須)]** テキスト ボックスの横にある **[参照]** を選択します。
1. **[ストレージ アカウント]** ページで、`Install_IIS.ps1` スクリプトをアップロードしたストレージ アカウントの名前を選択し、**[コンテナー]** ページで **[scripts]** を選択し、**[scripts]** ページで **[Install_IIS.ps1]** を選択し、**[選択]** をクリックします。
1. **[Windows 用カスタム スクリプト拡張機能の構成 - Azure Arc 拡張機能]** ページに戻り、**[確認と作成]**、**[作成]** の順に選択します。

    >**注**:これにより、サーバー arclab-winvm1 でスクリプトが自動的に実行されます。 スクリプトが完了するのを待ちます。 これには 3 分ほどかかります。

1. スクリプトが正常に完了したことを確認するには、Web ブラウザー ウィンドウで別のタブを開き、[http:\\localhost](http:\\localhost) に移動します。 ページに Web サーバーの名前 (この場合は `arclab-winvm1`) が表示されていることを確認します。 または、`C:\inetpub\wwwroot\iisstart.htm` の内容を確認することもできます。 

#### タスク 2:azcmagent CLI を使用して Azure Arc 対応 Windows サーバーを構成する

このタスクでは、azcmagent CLI を使用して、このラボで先ほど Azure Arc にオンボードした Windows サーバーのいずれかにレガシ Azure Log Analytics エージェント (Microsoft Monitoring Agent とも呼ばれます) がインストールされるのをブロックします。 

>**注**:Azure Connected Machine エージェント コマンド ライン ツールは、azcmagent.exe (azcmagent CLI とも呼ばれます) として実装され、Azure Arc 対応サーバーの構成、管理、トラブルシューティングに使用できます。 このツールは Azure Connected Machine エージェントと共にインストールされ、それが実行されているサーバーに固有のアクションを制御します。 このツールの詳細については、「[azcmagent CLI リファレンス](https://learn.microsoft.com/en-us/azure/azure-arc/servers/azcmagent)」を参照してください。

1. **arclab-winvm1 へのリモート デスクトップ セッション内で、** スタート メニューから **[Windows System]** フォルダーを開き、**[コマンド プロンプト]** を選択します。
1. コマンド プロンプトから、次のコマンドを実行して、Azure Arc Connected Machine エージェントの現在の構成設定を一覧表示します。

    ```cmd
    azcmagent config list
    ```

1. 出力を確認し、次の内容が含まれていることを確認します。

    ```cmd
    Local Configuration Settings
      incomingconnections.enabled (preview)             : true
      incomingconnections.ports (preview)                   : []
      connection.type (preview)                             : direct
      proxy.url                                             :
      proxy.bypass                                          : []
      extensions.allowlist                                  : []
      extensions.blocklist                                  : []
      guestconfiguration.enabled                            : true
      extensions.enabled                                    : true
      config.mode                                           : full
      guestconfiguration.agent.cpulimit                     : 5
      extensions.agent.cpulimit                             : 5
    ```

1. 次のコマンドを実行して、Microsoft Monitoring Agent のインストールをブロックします。

    ```cmd
    azcmagent config set extensions.blocklist "Microsoft.EnterpriseCloud.Monitoring/MicrosoftMonitoringAgent"
    ```

1. `azcmagent config list` コマンドを再実行し、その出力を確認し、次の内容が含まれていることを確認します。

    ```cmd
    Local Configuration Settings
      incomingconnections.enabled (preview)             : true
      incomingconnections.ports (preview)                   : []
      connection.type (preview)                             : direct
      proxy.url                                             :
      proxy.bypass                                          : []
      extensions.allowlist                                  : []
      extensions.blocklist                                  : [Microsoft.EnterpriseCloud.Monitoring/MicrosoftMonitoringAgent]
      guestconfiguration.enabled                            : true
      extensions.enabled                                    : true
      config.mode                                           : full
      guestconfiguration.agent.cpulimit                     : 5
      extensions.agent.cpulimit                             : 5
    ```

#### タスク 3:ラボ環境のプロビジョニングを解除する

このタスクでは、追加料金を回避するために、このラボで作成したすべての Azure リソースを削除します。 

1. ラボ コンピューターに戻り、Azure portal が表示されている Web ブラウザーで、Cloud Shell PowerShell セッションを起動します。 
1. Cloud Shell ペインの PowerShell セッションで、次のコマンドを実行して、ラボ全体で作成されたリソース グループ内のすべてのリソースを削除します。

    ```powershell
    Get-AzResourceGroup -Name 'arclab-infra-RG' | Remove-AzResourceGroup -Force -AsJob
    Get-AzResourceGroup -Name 'arclab-servers-RG' | Remove-AzResourceGroup -Force -AsJob
    ```

    >**注**:コマンドは非同期的 (-AsJob パラメーターによって決定) に実行されます。そのため、含まれているリソース グループとリソースが実際に削除されるまでに数分かかります。
