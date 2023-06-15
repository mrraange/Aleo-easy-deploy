# Aleo-easy-deploy
1. Create a new Aleo wallet, or use an existing one.
Go to the website and click the "Create" button. Save the output data, be vigilant, don't lose it
https://aleo.tools/
2. Request tokens for your wallet using your mobile phone number:
Send 50 credits to aleo... (previously received address)

3. Download the packages and create a tmux session
sudo apt update && \
sudo apt install make clang pkg-config libssl-dev build-essential gcc xz-utils git curl vim tmux ntp jq llvm ufw -y && \
tmux new -s deploy

4. Add wallet and private key as a variable.
echo Enter your Private Key: && read PK && \
echo Enter your View Key: && read VK && \
echo Enter your Address: && read ADDRESS

echo Enter your Transaction ID: && read TI
CIPHERTEXT=$(curl -s https://vm.aleo.org/api/testnet3/transaction/$TI | jq -r '.execution.transitions[0].outputs[0].value')

5. Install software
cd $HOME
git clone https://github.com/AleoHQ/snarkOS.git --depth 1
cd snarkOS
bash ./build_ubuntu.sh
source $HOME/.bashrc
source $HOME/.cargo/env

cd $HOME
git clone https://github.com/AleoHQ/leo.git
cd leo
cargo install --path .

6. Deploy a contract
NAME=helloworld_"${ADDRESS:4:6}"
mkdir $HOME/leo_deploy
cd $HOME/leo_deploy
leo new $NAME

RECORD=$(snarkos developer decrypt --ciphertext $CIPHERTEXT --view-key $VK)

snarkos developer deploy "$NAME.aleo" \
--private-key "$PK" \
--query "https://vm.aleo.org/api" \
--path "$HOME/leo_deploy/$NAME/build/" \
--broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" \
--fee 4000000 \
--record "$RECORD"

7. Execute a contract

Use the hash as the answer for the following command.

echo Enter your Deploy hash: && read DH

CIPHERTEXT=$(curl -s https://vm.aleo.org/api/testnet3/transaction/$DH | jq -r '.fee.transition.outputs[].value')

RECORD=$(snarkos developer decrypt --ciphertext $CIPHERTEXT --view-key $VK)

snarkos developer execute "$NAME.aleo" "hello" "1u32" "2u32" \
--private-key $PK \
--query "https://vm.aleo.org/api" \
--broadcast "https://vm.aleo.org/api/testnet3/transaction/broadcast" \
--fee 1000000 \
--record "$RECORD"

Finish!!

