# bruteforce 

import random
import string
import hashlib
import time
import json
import os
from cryptography.fernet import Fernet

# Generate a strong password
def generate_password(length=16):
    characters = string.ascii_letters + string.digits + string.punctuation
    password = ''.join(random.choice(characters) for _ in range(length))
    return password

# Brute-force attack simulation
def brute_force_attack(target_password):
    characters = string.ascii_letters + string.digits + string.punctuation
    attempt = ""
    start_time = time.time()
    
    for length in range(1, len(target_password) + 1):
        for attempt in ("".join(i) for i in itertools.product(characters, repeat=length)):
            if attempt == target_password:
                end_time = time.time()
                print(f"Password cracked: {attempt} in {end_time - start_time:.2f} seconds")
                return
    print("Failed to crack the password")

# Password manager
class PasswordManager:
    def __init__(self, key_file="key.key", db_file="passwords.json"):
        self.db_file = db_file
        self.key = self.load_key(key_file)
        self.cipher = Fernet(self.key)
        self.passwords = self.load_passwords()

    def load_key(self, key_file):
        if os.path.exists(key_file):
            with open(key_file, "rb") as file:
                return file.read()
        key = Fernet.generate_key()
        with open(key_file, "wb") as file:
            file.write(key)
        return key
    
    def load_passwords(self):
        if os.path.exists(self.db_file):
            with open(self.db_file, "rb") as file:
                encrypted_data = file.read()
                return json.loads(self.cipher.decrypt(encrypted_data).decode())
        return {}
    
    def save_passwords(self):
        encrypted_data = self.cipher.encrypt(json.dumps(self.passwords).encode())
        with open(self.db_file, "wb") as file:
            file.write(encrypted_data)
    
    def add_password(self, site, password):
        self.passwords[site] = password
        self.save_passwords()
    
    def get_password(self, site):
        return self.passwords.get(site, "No password found")

if __name__ == "__main__":
    pm = PasswordManager()
    site = "example.com"
    password = generate_password()
    print(f"Generated password for {site}: {password}")
    pm.add_password(site, password)
    print(f"Retrieved password: {pm.get_password(site)}")
