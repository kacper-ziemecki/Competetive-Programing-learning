Dodawanie big int w c++ (3 - 750):

//
// Created by NSC on 11/7/18.
//
#include <string.h>
#include <cstring>
#include "LargeIntegers.h"

unsigned long InfInt::length() const{
    if (value == "0")
    return 0;
    return value.length();
}

InfInt::InfInt(){
    value = "0";
}

InfInt::InfInt(const std::string &s) {
    unsigned long start = s.find_first_not_of("0-");

    if (!std::strcmp(&s[start],"")) {
    std::strcpy(&value[0], "0");
    }

    if(s[0] == '-') {
    sign = 1;
    }
    value = &s[start];
}

InfInt::InfInt(long int a) {
    sign =0;
    if (a < 0){
    sign = 1;
    a *= -1;
    }

    value = std::to_string(a);
}

InfInt InfInt::operator=(long int a){
    return *new InfInt(a);
}

std::string InfInt::getValue() const{
    return value;
}

bool InfInt::getSign() const{
    return sign;
}

void InfInt::setSign(const bool s){
    sign = s;
}

InfInt::operator int() const{
    int final = 0 ;
    int pow = 1;
    for (long i = length() - 1 ; i >= 0; i--) {
    final +=  pow * (value[i] - '0');
    pow*= 10;
    }
    if (sign)
    final *= -1;
    return final;
}

std::string InfInt::getBinary() const{
    //if the number is 0
    if (!length())
    return "0";
    InfInt temp = *new InfInt(value);
    //sign at the start and then from lsb to msb
    std::string result = "";
    result = result + (char)((int)sign + '0');
    std::string current;

    while (temp > 0){
    current=  (temp%2).getValue();
    if (current.length() == 0)
        result = result + "0";
    else
        result = result + current;
    temp = temp/2;
    }

    return result;
}

void InfInt::initFromBinary(const std::string b){
    sign = b[0] - '0';
    InfInt temp = 0;
    InfInt shift = 1;
    for (long i=1; i<b.length(); i++){
    temp += *new InfInt((int)(b[i] - '0')) * shift;
    shift = shift * 2;
    }

    value = temp.value;
}

std::ostream& operator<<(std::ostream& os, const InfInt& a){
    if (a.length() == 0 || a.getValue()[0] == '0') {
    std::string zero = "0";
    return os << zero;
    }
    if (a.getSign())
    return os << "-" + a.getValue();
    else
    return os<< a.getValue();
}

std::string InfInt::align(const unsigned long l) const{
    std::string newStr = value;
    for (unsigned long i=0; i<l; i++){
    newStr = "0" + newStr;
    }
    return newStr;
}

InfInt InfInt::alignLeft(const unsigned long l ) const{
    std::string newStr = value;
    for (unsigned long i=0; i<l; i++){
    newStr = newStr + "0";
    }
    return newStr;
}

void InfInt::operator++(){
    *this+=1;
}

void InfInt::operator--(){
    *this= *this - 1;
}

//adding the values of the input w/o their sign
std::string InfInt::add(const InfInt & a) const {
    unsigned long numOfZeros = value.length() - a.length();
    std::string aligned = a.align(numOfZeros);
    //now this and a are in the same length

    std::string result;
    int x = 0;
    bool carry = 0;

    for (long i = value.length() -1; i>=0; i--){
    //according to the ascii table '0' = 0 + 48 so subtracting 48 is a more sufficient way to convert
    x=value[i] + aligned[i] - 96 + carry;
    carry = x/10;
    result = (char)(x%10 + 48) + result;
    }

    //in case of "overflow"
    if (carry)
    result = "1" + result;

    return result;
}

InfInt InfInt::operator+(const InfInt & a) const{
    //a + (-b) = a - b
    if (!getSign() && a.getSign()) {
    InfInt * temp = new InfInt(a.value);
    return *this - *temp;
    }

    //(-a) + b = b - a
    if (getSign() && !a.getSign()) {
    //temp is positive. temp = -a
    InfInt * temp = new InfInt(value);
    return a - *temp;
    }

    //a + b = b + a
    if (value.length() < a.length())
    return a.operator+(*this);

    std::string result = this->add(a);

    //(-a) + (-b) = -(a + b)
    if (getSign() && a.getSign())
    return *new InfInt('-' + result);

    return *new InfInt(result);
}

//when this function is called, we can be sure that a > b
std::string InfInt::subtract(const InfInt& a) const{

    if (a.length() > length())
    return a.subtract(*this);
    long numOfZeros = this->length() - a.length();
    std::string aligned = a.align(numOfZeros);
    //now this and a are in the same length

    std::string result = "";
    int x = 0;
    bool carry = 0;

    for (long i = value.length() -1; i>=0; i--){
    x=value[i] - aligned[i] - carry;
    carry = false;
    if (x < 0){
        x+=10;
        carry = true;
    }
    result = (char)(x + 48) + result;
    }

    return result;
}

InfInt InfInt::operator-(const InfInt &a) const{
    if (length() == 0)
    return a * *new InfInt(-1);
    if (a.length() == 0)
    return *this * *new InfInt(-1);
    //both positive
    if (!getSign() && !a.getSign()) {
    if (*this > a)
        return *new InfInt(this->subtract(a));

    // a - b = -(b - a)
    InfInt * result = new InfInt(a.subtract(*this));
    result->setSign(true);
    return *result;
    }

    //a - (-b) = a + b
    if (!getSign() && a.getSign())
    return *new InfInt(this->add(a));

    //(-a) - b = - (a + b)
    if (getSign() && !a.getSign()) {
    InfInt result = this->add(a);
    result.setSign(true);
    return result;
    }

    //(-a) - (-b) = b - a
    InfInt temp1 = *new InfInt(value);
    InfInt temp2 = *new InfInt(a.value);
    return temp2 - temp1;
//    if (a > *this)
//        return *new InfInt(a.subtract(*this));
//
//    InfInt * result = new InfInt(this->subtract(a));
//    result->setSign(true);
//    return *result;
}

void InfInt::operator+=(const InfInt& a){
    InfInt temp = *this + a;
    value = temp.value;
    sign = temp.sign;
}

void InfInt::operator-=(const InfInt& a){
    InfInt temp = *this - a;
    value = temp.value;
    sign = temp.sign;
}

InfInt InfInt::operator*(const InfInt& a) const{

    InfInt final = 0;
    std::string result;
    InfInt* temp;


    int carry;
    int current;

    //fast mult algorithm. the same we were taught in elementary.
    for(long i=length() - 1;i >= 0; i--){
    carry = 0;
    result = "";
    for (long j=a.length() - 1; j >= 0; j--){
        current = (value[i] - '0') * (a.value[j] - '0') + carry;
        result = (char)(current % 10 + '0') + result;
        carry = current / 10;
    }

    if (carry > 0)
        result = (char)(carry + '0') + result;

    temp = new InfInt(result);
    final += *new InfInt(temp->alignLeft(length() - i - 1));
    }

    final.setSign(sign ^ a.sign);
    return final;
}

//long division implementation
InfInt InfInt::operator/(const InfInt& a) const {
    if (a.length() == 0 || a.getValue()[0] == '0') {
    throw "Devision By Zero";
    }

    //divider = |a|
    InfInt divider = *new InfInt(a.getValue());
    if (divider > *new InfInt(value))
    return *new InfInt();

    std::string result;
    int idx = 0;

    //temp is the part of the divided that's being currently focused
    InfInt temp = value[idx] - '0';
    while (temp < divider.value)
    temp = temp * 10 + (value[++idx] - '0');

    while(idx < length()) {

    if (temp == 0){
        result = result + "0";
        idx++;
    }
    else {
        // Find prefix of number that's larger
        // than a.value.


        InfInt multNum = 1;
        InfInt leftover = temp - divider;
        while (leftover >= divider) {
            leftover -= divider;
            multNum += 1;
        }

        leftover = temp - (multNum * divider);
        result = result + multNum.getValue();

        temp = leftover * 10 + (value[++idx] - '0');
        temp.setSign(false);
    }
    }
    // If a.value is greater than value
    if (result.length() == 0)
    return *new InfInt();

    InfInt final = *new InfInt(result);
    final.setSign(this->sign ^ a.getSign());
    return final;
}

InfInt InfInt::operator%(const InfInt& a) const {
    if (a.length() == 0 || a.getValue()[0] == '0') {
    throw "Modulo By Zero";
    }

    if (a > *this)
    return *new InfInt(*this);
    //divider = |a|
    InfInt divider = *new InfInt(a.getValue());

    if (divider > *new InfInt(value))
    return (*this + a) % a;
    std::string result;
    int idx = 0;
    InfInt leftover;

    InfInt temp = value[idx] - '0';
    while (temp < divider.value)
    temp = temp * 10 + (value[++idx] - '0');

    while(idx < length()) {
    // Find prefix of number that's larger
    // than a.value.


    InfInt multNum = 0;
    leftover = temp;
    while (leftover >= divider) {
        leftover -= divider;
        multNum += 1;
    }

    leftover = temp - (multNum * divider);
    leftover.setSign(false);

    temp = leftover * 10 + (value[++idx] - '0');
    }
    // If a.value is greater than value

    return leftover;
}

bool InfInt::operator<(const InfInt& a) const{
    if (getSign() && !a.getSign())
    return true;
    if (!getSign() && a.getSign())
    return false;

    unsigned long l1 = length(), l2 = a. length();

    //both positive
    if (!getSign() && !a.getSign()) {
    if (l1 > l2)
        return false;

    if (l1 < l2)
        return true;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return false;

        if (value[i] < a.value[i])
            return true;
    }
    }

    else {
    if (l1 > l2)
        return true;

    if (l1 < l2)
        return false;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return true;

        if (value[i] < a.value[i])
            return false;
    }
    }

    //equal
    return false;
}

bool InfInt::operator<=(const InfInt& a) const{
    if (getSign() && !a.getSign())
    return true;
    if (!getSign() && a.getSign())
    return false;

    unsigned long l1 = length(), l2 = a. length();

    //both positive
    if (!getSign() && !a.getSign()) {
    if (l1 > l2)
        return false;

    if (l1 < l2)
        return true;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return false;

        if (value[i] < a.value[i])
            return true;
    }
    }

    else {
    if (l1 > l2)
        return true;

    if (l1 < l2)
        return false;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return true;

        if (value[i] < a.value[i])
            return false;
    }
    }

    //equal
    return true;
}

bool InfInt::operator>(const InfInt& a) const{
    if (getSign() && !a.getSign())
    return false;
    if (!getSign() && a.getSign())
    return true;

    unsigned long l1 = length(), l2 = a. length();

    //both negative
    if (getSign() && a.getSign()) {
    if (l1 > l2)
        return false;

    if (l1 < l2)
        return true;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return false;

        if (value[i] < a.value[i])
            return true;
    }
    }

    else {
    if (l1 > l2)
        return true;

    if (l1 < l2)
        return false;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return true;

        if (value[i] < a.value[i])
            return false;
    }
    }

    //equal
    return false;
}

bool InfInt::operator>=(const InfInt& a) const{
    if (getSign() && !a.getSign())
    return false;
    if (!getSign() && a.getSign())
    return true;

    unsigned long l1 = length(), l2 = a. length();

    //both negative
    if (getSign() && a.getSign()) {
    if (l1 > l2)
        return false;

    if (l1 < l2)
        return true;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return false;

        if (value[i] < a.value[i])
            return true;
    }
    }

    else {
    if (l1 > l2)
        return true;

    if (l1 < l2)
        return false;

    for (long i = 0; i < l1; i++) {
        if (value[i] > a.value[i])
            return true;

        if (value[i] < a.value[i])
            return false;
    }
    }

    //equal
    return true;
}

bool InfInt::operator==(const InfInt& a) const{

    if (length() == 0 && a.length() == 0)
    return true;

    //if the signs are not the same
    if (getSign() ^ a.getSign())
    return false;

    if (length() != a.length())
    return false;

    for(long i = 0; i < length(); i++){
    if (value[i] != a.value[i])
        return false;
    }

    return true;
}

void InfInt::operator<<=(const InfInt& a){

    InfInt result = *this << a;

    value = result.getValue();
    sign = result.getSign();
}

void InfInt::operator>>=(const InfInt& a){

    InfInt result = *this >> a;

    value = result.getValue();
    sign = result.getSign();
}

void InfInt::operator&=(const InfInt& a){
     InfInt temp = *this & a;
     value = temp.getValue();
     sign = temp.getSign();
}

InfInt InfInt::operator^(const InfInt& a) const{
    std::string b1 = this->getBinary();
    std::string b2 = a.getBinary();

    long l1 = b1.length();
    long l2 = b2.length();

    //adding zeros from right doesn't change the value of the number
    int numOfZeros;

    if (l1 > l2) {
    numOfZeros = l1- l2;
    for (int i=0; i < numOfZeros; i++)
        b2 = b2 + "0";
    }
    else if (l2 > l1) {
    numOfZeros = l2- l1;
    for (int i=0; i < numOfZeros; i++)
        b1 = b1 + "0";
    }

    std::string result = "";

    for (int i = 0; i<l1+1; i++){
    if (b1[i] == '1' ^ b2[i] == '1')
        result = result + "1";
    else
        result = result + "0";
    }

    InfInt final = *new InfInt();
    final.initFromBinary(result);
    return final;
}

InfInt InfInt::operator|(const InfInt& a) const{
    std::string b1 = this->getBinary();
    std::string b2 = a.getBinary();

    long l1 = b1.length();
    long l2 = b2.length();

    //adding zeros from right doesn't change the value of the number
    int numOfZeros;

    if (l1 > l2) {
    numOfZeros = l1- l2;
    for (int i=0; i < numOfZeros; i++)
        b2 = b2 + "0";
    }
    else if (l2 > l1) {
    numOfZeros = l2- l1;
    for (int i=0; i < numOfZeros; i++)
        b1 = b1 + "0";
    }

    std::string result = "";

    for (int i = 0; i<l1; i++){
    if (b1[i] == '1' || b2[i] == '1')
        result = result + "1";
    else
        result = result + "0";
    }

    InfInt final = *new InfInt();
    final.initFromBinary(result);
    return final;
}

InfInt InfInt::operator&(const InfInt& a) const {
    std::string b1 = this->getBinary();
    std::string b2 = a.getBinary();

    long l1 = b1.length();
    long l2 = b2.length();

    //adding zeros from right doesn't change the value of the number
    int numOfZeros;

    if (l1 > l2) {
    numOfZeros = l1- l2;
    for (int i=0; i < numOfZeros; i++)
        b2 = b2 + "0";
    }
    else if (l2 > l1) {
    numOfZeros = l2- l1;
    for (int i=0; i < numOfZeros; i++)
        b1 = b1 + "0";
    }

    std::string result = "";

    for (int i = 0; i<l1; i++){
    if (b1[i] == '1' && b2[i] == '1')
        result = result + "1";
    else
        result = result + "0";
    }

    InfInt final = *new InfInt();
    final.initFromBinary(result);
    return final;
}

InfInt InfInt::operator<<(const InfInt& a) const {
    std::string b = this->getBinary();
    std::string result = "";
    result = result + b[0];

    //adding 0's multiplies by 2
    for (InfInt i = 0; i < a; i+=1){
    result = result + "0";
    }

    //adding the original bits after adding zeros
    for (long j = 1 ; j < b.length(); j ++){
    result = result + b[j];
    }

    InfInt final = *new InfInt();
    final.initFromBinary(result);
    return final;
}

InfInt InfInt::operator>>(const InfInt& a) const {
    std::string b = this->getBinary();

    b.erase(1, (int) a);

    InfInt final = *new InfInt();
    final.initFromBinary(b);
    return final;
}

c++ implementation for binary search tree(751 - 870):

#include <iostream>

/* Node Structure */
struct BstNode
{
    int data;
    BstNode *left;
    BstNode *right;
};

/* Create New Node */
BstNode *createNode(int data)
{
    BstNode *newNode = new BstNode();

    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

/* Insert Node in Binary Search Tree */
BstNode *insertNode(BstNode *root, int data)
{
    if (root == NULL)
        root = createNode(data);
    else if (data <= root->data)
        root->left = insertNode(root->left, data);
    else
        root->right = insertNode(root->right, data);
    return root;
}

/* Search Node Value in Binary Search Tree */
bool searchNode(BstNode *root, int data)
{
    if (root == NULL)
        return false;
    else if (data == root->data)
        return true;
    else if (data <= root->data)
        return searchNode(root->left, data);
    else
        return searchNode(root->right, data);
}

/* Find Minimal Node Value in Binary Search Tree */
BstNode *findMinNode(BstNode *root)
{
    while (root->left != NULL)
        root = root->left;
    return root;
}

/* Delete Node in Binary Search Tree */
BstNode *deleteNode(BstNode *root, int data)
{
    if (root == NULL)
        return root;
    else if (data < root->data)
        root->left = deleteNode(root->left, data);
    else if (data > root->data)
        root->right = deleteNode(root->right, data);
    else
    {
        if (root->left == NULL && root->right == NULL)
        {
            delete root;
            root = NULL;
        }
        else if (root->left == NULL)
        {
            BstNode *temp = root;
            root = root->right;
            delete temp;
        }
        else if (root->right == NULL)
        {
            BstNode *temp = root;
            root = root->left;
            delete temp;
        }
        else
        {
            BstNode *min = findMinNode(root->right);
            root->data = min->data;
            root->right = deleteNode(root->right, min->data);
        }
    }
    return root;
}

int main()
{
    BstNode *root = NULL;

    /* Insert Node with Value */
    root = insertNode(root, 12);
    root = insertNode(root, 15);
    root = insertNode(root, 5);
    root = insertNode(root, 53);
    root = insertNode(root, 98);
    root = insertNode(root, 0);

    /* Search Node by Value */
    std::cout << searchNode(root, 53) << std::endl;
    std::cout << searchNode(root, 42) << std::endl;
    std::cout << searchNode(root, 15) << std::endl;

    /* Delete Node by Value */
    root = deleteNode(root, 53);
    root = deleteNode(root, 15);

    /* Search Node by Value */
    std::cout << searchNode(root, 53) << std::endl;
    std::cout << searchNode(root, 15) << std::endl;
    return 0;
}

red black tree implementation using c++(875 - 1160):

#ifndef _RED_BLACK_ //protection from multiple inclusion
#define _RED_BLACK_
struct NODE
{
	int key;
	int color; //black = 0 and red = 1
	NODE* p; //pointer to parent
	NODE* r; //pointer to right child
	NODE* l; //pointer to left child
};
NODE NULL_NODE = {-1,0,0,0,0}; //sentinal
class RBT //assumed no equal keys
{
	private:
		NODE* ROOT;
	public:
		RBT()
		{
			ROOT = 0; //initally points to nothing
		}
		NODE* search(int k) const //search for node with key k
		{
			if(ROOT == 0) return &NULL_NODE; //no tree
			NODE* CURRENT = ROOT;
			for(;;)
			{
				if(k == CURRENT->key) return CURRENT;
				else if(k < CURRENT->key)
				{
					if(CURRENT->l == 0) return &NULL_NODE; //no node with given key
					else CURRENT = CURRENT->l;
				}
				else if(k > CURRENT->key)
				{
					if(CURRENT->r == 0) return &NULL_NODE; //no node with given key
					else CURRENT = CURRENT->r;
				}
			}
		}
		NODE* minimum(NODE& N) const
		{
			if(ROOT == 0) return &NULL_NODE; //no tree
			NODE* CURRENT = search(N.key); //pointer to sub-tree root
			while(CURRENT->l != 0) CURRENT = CURRENT->l; //find minimun
			return CURRENT;
		}
		bool insert_node(int k) //true if successful, false otherwise.
		{
			NODE* insert = new NODE;
			//using new instead of just declaring as NODE insert because memory can be freed
			//in delete_node and insert->key looks more intuitive (well, to me anyway) than insert.key
			insert->key = k;
			insert->color = 1; //red is default
			insert->p = 0; //if it is the root, parent is null pointer
			insert->l = 0; /*	So that leaves will have	*/
			insert->r = 0; /*	null pointers for children	*/
			if(ROOT == 0) //first insert
			{
				ROOT = insert;
				return true;
			}
			for(;;)
			{
				if(k == insert->key) break;
				else if(k < insert->key)
				{
					if(insert->l != 0) insert = insert->l;
					else 
					{
						insert->l = insert;
						insert->p = insert;
						break;
					}
				}
				else if(k > insert->key)
				{
					if(insert->r != 0) insert = insert->r;
					else
					{
						insert->r = insert;
						insert->p = insert;
						break;
					}
				}
			}
			//fixing up violations
			while(insert->p != ROOT && insert->p->color == 1)
			{
				if(insert->p == insert->p->p->l) // insert is in the left subtree
				{
					NODE* aunt = insert->p->p->r; // chose aunt insted of uncle because aunt has one
					if(aunt->color == 1)	// character less than uncle. I am not sexist.
					{
						//case 1
						insert->color = 0;
						aunt->color = 0;
						insert->p->p->color = 1;
						insert = insert->p->p;
						continue;
					}
					else if(insert = insert->p->p->r)
					{
						//case 2
						left_rotate(insert->p->key); //rotations defined below
					}
					else
					{
						//case 3
						insert->p->color = 0;	// recoloring first and rotating next because
						insert->p->p->color = 1; // rotating changes parent-child relations
						right_rotate(insert->p->p->key);
						return true;
					}
				}
				else // insert is in the right subtree
				{
					// same code with l interchanged with r. Copy-pasted (you can prolly tell by the cringy comment in the next line)
					NODE* aunt = insert->p->p->l; // chose aunt insted of uncle because aunt has one
					if(aunt->color == 1)		  // character less than uncle. I am not a sexist.
					{
						//case 1
						insert->color = 0;
						aunt->color = 0;
						insert->p->p->color = 1;
						insert = insert->p->p;
						continue;
					}
					else if(insert = insert->p->p->l)
					{
						//case 2
						right_rotate(insert->p->key); //rotations defined below
					}
					else
					{
						//case 3
						insert->p->color = 0;	// recoloring first and rotating next because
						insert->p->p->color = 1; // rotating changes parent-child relations
						left_rotate(insert->p->p->key);
						return true;
					}
				}
			}
			return false; //couldn't insert
		}
		void transplant(NODE* u, NODE* v)
		{
			if(u->p == 0) ROOT = v; // u is the root
			else if(u == u->p->l) u->p->l = v; // u is a left child
			else u->p->r = v; // u is a right child
			v->p = u->p;
		}
		bool delete_node(int k) //true if successful, false otherwise.
		{
			NODE* del = search(k); //search for the node to be deleted
			int original_color = del->color;
			NODE* y = del;
			NODE* x; // x becomes either y's right child or 0
			if(del->l == 0) // no left child
			{
				x = del->r; 
				transplant(del,del->r);
			}
			else if(del->r == 0) // no right child
			{
				x = del->l;
				transplant(del,del->l);
			}
			else // both children
			{
				y = minimum(*del->r); // successor
				original_color = y->color;
				x = y->r;
				if(y->p->key == del->key) x->p == y; //del's right child turned out to be its successor
				else
				{
					transplant(y,y->r);
					y->r = del->r;
					y->r->p = y;
				}
				y->l = del->l;
				y->l->p = y;
				y->color = del->color;
			}
			//fixing up violations
			if(original_color == 0)
			{
				NODE* aunt;
				while(x != ROOT && x->color == 0)
				{
					if(x->p->l) //x a the left child
					{
						aunt = x->p->r;
						if(aunt->color == 1)
						{
							// case 1
							aunt->color = 0;
							x->p->color = 1;
							left_rotate(x->p->key);
							aunt = x->p->r;
						}
						if(aunt->l->color == 0 && aunt->r->color == 0)
						{
							//case 2
							aunt->color = 1;
							x = x->p;
						}
						else if(aunt->r->color == 0)
						{
							//case 3
							aunt->l->color = 0;
							aunt->color = 1;
							right_rotate(aunt->key);
							aunt = x->p->r;
						}
						//case 4
						aunt->color = x->p->color;
						x->p->color = 0;
						aunt->r->color = 0;
						left_rotate(x->p->key);
						x = ROOT;
					}
					// interchange l and r
					else
					{
						aunt = x->p->l;
						if(aunt->color == 1)
						{
							// case 1
							aunt->color = 0;
							x->p->color = 1;
							left_rotate(x->p->key);
							aunt = x->p->l;
						}
						if(aunt->r->color == 0 && aunt->l->color == 0)
						{
							//case 2
							aunt->color = 1;
							x = x->p;
						}
						else if(aunt->l->color == 0)
						{
							//case 3
							aunt->r->color = 0;
							aunt->color = 1;
							left_rotate(aunt->key);
							aunt = x->p->l;
						}
						//case 4
						aunt->color = x->p->color;
						x->p->color = 0;
						aunt->l->color = 0;
						left_rotate(x->p->key);
						x = ROOT;
					}
				}
				x->color = 0;
			}
			return false; //couldn't delete
		}
		void left_rotate(int k)
		{
			NODE* x = search({k}); //node to be rotated
			NODE* y = x->r; //set y
			x->r = y->l; //y's left subtree is x's right subtree
			if(y->l != 0) y->l->p = x;
			y->p = x->p; //linking parent
			if(x->p == 0) ROOT = y;
			else if(x == x->p->l) x->p->l =y;
			else x->p->r = y;
			y->l = x; //put x on y's left
			x->p = y;
		}
		// same with x and y interchanged
		void right_rotate(int k)
		{
			NODE* x = search({k}); //node to be rotated
			NODE* y = x->l; //set y
			x->l = y->r; //y's right subtree is x's left subtree
			if(y->r != 0) y->r->p = x;
			y->p = x->p; //linking parent
			if(x->p == 0) ROOT = y;
			else if(x == x->p->r) x->p->r =y;
			else x->p->l = y;
			y->r = x; //put x on y's right
			x->p = y;
		}
};
#endif
