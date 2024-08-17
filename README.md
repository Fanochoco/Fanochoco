import json
import argparse
from cryptography.fernet import Fernet

# Générer une clé de chiffrement
def generate_key():
    key = Fernet.generate_key()
    with open("secret.key", "wb") as key_file:
        key_file.write(key)

# Charger la clé de chiffrement
def load_key():
    return open("secret.key", "rb").read()

# Chiffrer un mot de passe
def encrypt_password(password):
    key = load_key()
    f = Fernet(key)
    encrypted_password = f.encrypt(password.encode())
    return encrypted_password

# Déchiffrer un mot de passe
def decrypt_password(encrypted_password):
    key = load_key()
    f = Fernet(key)
    decrypted_password = f.decrypt(encrypted_password).decode()
    return decrypted_password

# Sauvegarder un mot de passe chiffré dans un fichier JSON
def save_password(account, encrypted_password):
    passwords = load_passwords()
    passwords[account] = encrypted_password.decode()
    with open("passwords.json", "w") as file:
        json.dump(passwords, file)

# Charger les mots de passe depuis un fichier JSON
def load_passwords():
    try:
        with open("passwords.json", "r") as file:
            passwords = json.load(file)
    except FileNotFoundError:
        passwords = {}
    return passwords

# Interface en ligne de commande
def main():
    parser = argparse.ArgumentParser(description="Gestionnaire de mots de passe")
    parser.add_argument("action", choices=["add", "get"], help="Ajouter ou récupérer un mot de passe")
    parser.add_argument("account", help="Le nom du compte (par ex. Facebook)")

    args = parser.parse_args()

    if args.action == "add":
        password = input("Entrez le mot de passe à stocker : ")
        encrypted = encrypt_password(password)
        save_password(args.account, encrypted)
        print(f"Mot de passe pour {args.account} enregistré.")
    elif args.action == "get":
        passwords = load_passwords()
        if args.account in passwords:
            decrypted = decrypt_password(passwords[args.account].encode())
            print(f"Mot de passe pour {args.account} : {decrypted}")
        else:
            print(f"Aucun mot de passe trouvé pour {args.account}.")

if __name__ == "__main__":
    generate_key()  # À exécuter une seule fois pour générer et stocker la clé
    main()
