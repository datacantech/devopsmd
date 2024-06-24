## когда нужно установить ansible
```
python3 -m pip download -d /path/fold ansible #если одинаковый питон на машине с которой качаем и где ставим
#если просто нужен ансиблpython3 -m pip download --only-binary=:all: --python-version 311 -d /mnt/c/work/forans311 ansible
python3 -m pip list
#если нужен конкретный ансибл для кубернетес, то получаем файлы requirements.txt создав и зайдя в папкуpython3 -m pip download --only-binary=:all: --python-version 311 -d /
mnt/c/work/ansible/forans760 -r requirements.txt#устанавливаем так, выполняем перед стартом cd /mnt/c/work/ansible/forans760
python3 -m pip install ansible --no-index --find-links file:///mnt/c/work/ansible/forans760
## установка гит из tar.gz
```
#ставим зависимостиtar -zxf git-2.35.2.tar.gz
```
cd git-2.35.2make configure
./configure --prefix=/usrmake all doc info
sudo make install install-doc install-html install-info
```
## если без вариантов и работаем из под винды
```
включи виртуализацию в BIOS (везде по разному)Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatformwsl --update --web-download # опция --web-download нужна здесь из-за ограничений на store
wsl --install -d ubuntu --web-download #установит крайнюю версиюwsl --install -d Ubuntu-18.04 --web-download #ставит 18.04 
wsl --install -d Ubuntu-22.04 --web-download 
если возникли проблемыsfc /scannow
DISM /Online /Cleanup-Image /RestoreHealthDISM /Online /Cleanup-Image /CheckHealth
DISM /Online /Cleanup-Image /ScanHealthchkdsk C: /scan /forceofflinefix
выкл/вкл платформа виртуализации в комонентах windows (с перезагрузкой)
```