import json
import os
from cryptography.fernet import Fernet

class PyVault:
    def __init__(self, key_file="secret.key", data_file="vault.json"):
        self.key_file = key_file
        self.data_file = data_file
        self.key = self.load_or_generate_key()
        self.cipher = Fernet(self.key)

    def load_or_generate_key(self):
        """Loads the existing key or creates a new one."""
        if os.path.exists(self.key_file):
            with open(self.key_file, "rb") as f:
                return f.read()
        else:
            key = Fernet.generate_key()
            with open(self.key_file, "wb") as f:
                f.write(key)
            return key

    def encrypt_data(self, data):
        return self.cipher.encrypt(data.encode()).decode()

    def decrypt_data(self, encrypted_data):
        return self.cipher.decrypt(encrypted_data.encode()).decode()

    def add_password(self, site, username, password):
        """Adds a new entry to the vault."""
        vault = self.load_vault()
        vault[site] = {
            "username": username,
            "password": self.encrypt_data(password)
        }
        self.save_vault(vault)
        print(f"✅ Credentials for '{site}' saved successfully.")

    def get_password(self, site):
        """Retrieves and decrypts a password."""
        vault = self.load_vault()
        if site in vault:
            user = vault[site]['username']
            pwd = self.decrypt_data(vault[site]['password'])
            print(f"🔑 Site: {site}\n👤 User: {user}\n🔓 Pass: {pwd}")
        else:
            print("❌ Site not found in vault.")

    def load_vault(self):
        if os.path.exists(self.data_file):
            with open(self.data_file, "r") as f:
                return json.load(f)
        return {}

    def save_vault(self, vault):
        with open(self.data_file, "w") as f:
            json.dump(vault, f, indent=4)

if __name__ == "__main__":
    vault = PyVault()
    
    print("--- PyVault Secure Manager ---")
    action = input("Choose: (1) Add Password (2) Get Password: ")
    
    if action == "1":
        s = input("Enter Site Name: ")
        u = input("Enter Username: ")
        p = input("Enter Password: ")
        vault.add_password(s, u, p)
    elif action == "2":
        s = input("Enter Site Name: ")
        vault.get_password(s)
