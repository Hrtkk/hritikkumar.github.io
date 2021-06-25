# hritikkumar.com

void mstrcpy(char *dest, char *src) {
	while (*src != '\0') *dest = *src, dest++, src++;
}

int mstrcmp(char* str1, char* str2) {
	while (*str1 != '\0' && *str2 != '\0' && *str1 == *str2)str1++, str2++;
	return *str1 - *str2;
}

class MailNode {
public:
	char subject[202];
	MailNode* next;
	MailNode* prev;
	MailNode* nextSimilar;

	MailNode() {
		subject[0] = '\0';
		next = nullptr;
		prev = nullptr;
		nextSimilar = nullptr;
	}

	MailNode(char sub[]) {
		mstrcpy(subject, sub);
		next = nullptr;
		prev = nullptr;
		nextSimilar = nullptr;
	}
};

class TrieNode {
public:
	char val;
	TrieNode* next[28];
	int counter;
	MailNode* mailPtr;

	TrieNode() {
		val = '#';
		for (int i = 0; i < 28; i++) next[i] = nullptr;
		counter = 0;
		mailPtr = nullptr;
	}

	TrieNode(char ch) {
		val = ch;
		for (int i = 0; i < 28; i++) next[i] = nullptr;
		counter = 0;
		mailPtr = nullptr;
	}
};

class Trie {
public:
	TrieNode* root;

	int getIndex(char ch) {
		if (ch == ' ') return 26;
		return ch - 'a';
	}

	int insert(char word[]) {
		TrieNode* head = root;
		int index = -1;
		for (int i = 0; word[i] != '\0'; i++) {
			index = getIndex(word[i]);
			if (head->next[index] == NULL) {
				head->next[index] = new TrieNode(word[i]);
				head = head->next[index];
			}
			else {
				head = head->next[index];
			}
		}
		head->counter++;
		return head->counter;
	}

	TrieNode* getTail(char word[]) {
		TrieNode* head = root;
		int index = -1;
		for (int i = 0; word[i] != '\0'; i++) {
			index = getIndex(word[i]);
			if (head->next[index] == NULL) {
				return nullptr;
			}
			else {
				head = head->next[index];
			}
		}
		return head;
	}

	int deleteWord(char word[]) {
		TrieNode* head = getTail(word);
		if (head == nullptr) return 0;
		int wordCount = head->counter;
		head->counter = 0;
		head->mailPtr = nullptr;
		return wordCount;
	}
};

class MailTrie : public Trie {
public:
	MailTrie(){}

	int insertMail(char subject[], TextTrie* textTrie, MailNode* node) {
		insert(subject);
		TrieNode* tail = getTail(subject);
		if (tail->mailPtr == nullptr) {
			tail->mailPtr = node;
		}
		else {
			node->nextSimilar = tail->mailPtr;
			tail->mailPtr = node;
		}
		Trie temp;
		char word[11];
		int wordIndex = 0;
		for (int i = 0; subject[i] != '\0'; i++) {
			if (subject[i] == ' ' || subject[i + 1] == '\0') {
				if (subject[i + 1] != '\0') {
					word[wordIndex++] = subject[i];
				}
				word[wordIndex] = '\0';
				if (temp.insert(word) == 1) textTrie->insertText(word);
			}
			else {
				word[wordIndex] = subject[i];
			}
		}
	}

	int deleteMail(char subject[], TextTrie* textTrie) {
		TrieNode* head = getTail(subject);
		int mCount = head->counter;
		head->counter = 0;
		Trie temp;
		char word[11];
		int wordIndex = 0;
		for (int i = 0; subject[i] != '\0'; i++) {
			if (subject[i] == ' ' || subject[i + 1] == '\0') {
				if (subject[i + 1] != '\0') {
					word[wordIndex++] = subject[i];
				}
				word[wordIndex] = '\0';
				if (temp.insert(word) == 1) textTrie->deleteText(word, mCount);
			}
			else {
				word[wordIndex] = subject[i];
			}
		}
		return mCount;
	}
};

class TextTrie: public Trie {
public:
	TextTrie(){}

	int insertText(char text[]) {
		insert(text);
		return getTail(text)->counter;
	}

	int deleteText(char text[], int cnt) {
		TrieNode* tail = getTail(text);
		tail->counter -= cnt;
	}
};

struct User {
	MailNode* frontMail;
	MailNode* backMail;
	int mailCount;
	int mailCapacity;
	MailTrie mailTrie;
	TextTrie textTrie;
public:
	User(int K) {
		mailCapacity = K;
		mailCount = 0;
		mailTrie = MailTrie();
		textTrie = TextTrie();
		frontMail = nullptr;
		backMail = nullptr;
	}

	void receiveMail(char subject[]) {
		MailNode* temp = new MailNode(subject);
		if (mailCount == 0) {
			frontMail = temp;
			backMail = frontMail;
			mailTrie.insertMail(subject, &textTrie, temp);
			mailCount++;
		}
		else {
			frontMail->next = temp;
			temp->prev = frontMail;
			frontMail = temp;
			if (mailCount == mailCapacity) {
				deleteMail(backMail->subject);
				backMail = backMail->next;
				backMail->prev = nullptr;
				mailTrie.insertMail(subject, &textTrie, temp);
			}
			else {
				mailTrie.insertMail(subject, &textTrie, temp);
				mailCount++;
			}
		}

	}

	int countMail() {
		return mailCount;
	}

	int deleteMail(char subject[]) {
		TrieNode* head = mailTrie.getTail(subject);
		int deletedMailCount = mailTrie.deleteMail(subject, &textTrie);
		if (head == nullptr) return 0;
		MailNode* mailHead = head->mailPtr;
		head->mailPtr = nullptr;
		while (mailHead) {
			if (mailHead->next != nullptr) {
				if (mailHead->prev != nullptr) {
					mailHead->prev->next = mailHead->next;
					mailHead->next->prev = mailHead->prev;
				}
				else {
					mailHead->next->prev = nullptr;
					backMail = mailHead->next;
				}
			}
			else {
				if (mailHead->prev == nullptr) {
					frontMail = nullptr;
					backMail = nullptr;
				}
				else {
					frontMail = mailHead->prev;
					frontMail->next = nullptr;
				}
			}
			mailCount--;
			MailNode* temp = mailHead;
			mailHead = mailHead->nextSimilar;
			temp->nextSimilar = nullptr;
		}
		return deletedMailCount;
	}

	int searchMail(char text[]) {
		return textTrie.getTail(text)->counter;
	}
};

User users[1002];

void init(int N, int K) {
	for (int i = 0; i <= N; i++) {
		users[i] = User(K);
	}
}

