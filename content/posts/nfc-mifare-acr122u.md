---
title: "How to authenticate and write data to Mifare Classic 1k using ACS ACR122U"
date: 2023-07-23T14:00:23+02:00
description: Write to Mifare Classic 1k using ACR122U.
draft: false
tags: [nfc, c++, programming, mifare]
---

## Goal

I bought some Mifare Classic 1k tags and all Android/iOS apps I tried (official NXP apps, NFC Tools Pro and Mifare Classic Tool) failed to write data to my tags! Mifare Classic tags are not as straight-forward as e.g. NTAG ones because they are proprietary (they do technically support NDEF, but it's not a compatible as you would like it to be). So to write data to these tags using my ACS ACR122U, I use the Windows Smart Card API to communicate with the NFC reader. Then I use special commands to authenticate the block I want to write data to, and then the data is written. I also added a function to NDEF format the entire card so that it becomes compatible with iOS and Android apps!

## Setup

To follow this post, you need:
* ACR122U
* Windows with Visual Studio
* Mifare Classic 1k Tag

## Let's get down to business

First, make sure that there are no special drivers for the ACR122U installed. If you followed my previous post where we used Zadig to replace the standard CCID driver to use WinUSB instead (requirement for nfcpy), then you need to go to the device manager and "Uninstall Device" to revert to CCID drivers. This requirement is taken from the ACR122U API documentation (2.2. Smart Reader Interface Overview). The default Windows drivers or the official ACS drivers work fine.

Now install Visual Studio (I use version 2022) and make sure when installing to also install the "Windows 11 SDK (10.0.22621.0)" so that you can access the winscard.h header. If you made your project in Visual Studio, make sure to right-click the project in the Solution Explorer, go to "Properties -> Linker -> Input" and in the field "Additional Dependencies" add ";winscard.lib". If you forget to do this, the compilation will fail. Also make sure to not accidently put 'winscard.h'.

Then get the code I wrote from my [Github repo](https://github.com/downIoads/nfc_mifaireClassic_acr122u_winscard/blob/main/main.c). To keep this post short, I won't go into many details, but basically the hard-coded byte values are taken from the documentations you need to read to understand what's happening. There are two important documents you should take the time to read:
* ACR122U API documentation (API-ACR122U-2.04.pdf)
* MIFARE Classic EV1 1K Product Sheet (MF1S50YYX_V1.pdf)

This way, you will understand which blocks are writable and why, what the keys A and B are, what the access conditions are etc. Anyways, here is my C++ code:

## C++

```c++
// thanks @gentilkiwi for reviewing my code and providing the base construct for the improved version!
#include <stdbool.h>
#include <stdio.h>
#include <stdlib.h>
#include <windows.h>
#include <winscard.h>

typedef struct _SCARD_DUAL_HANDLE {
	SCARDCONTEXT hContext;
	SCARDHANDLE hCard;
} SCARD_DUAL_HANDLE, * PSCARD_DUAL_HANDLE;


// allowed block values for safely writing data (assuming non-magic tag)
const BYTE allowedBlocks[] = {  0x01, 0x02, 0x04, 0x05, 0x06, 0x08, 0x09, 0x0A,
								0x0C, 0x0D, 0x0E, 0x10, 0x11, 0x12, 0x14, 0x15,
								0x16, 0x18, 0x19, 0x1A, 0x1C, 0x1D, 0x1E, 0x20,
								0x21, 0x22, 0x24, 0x25, 0x26, 0x28, 0x29, 0x2A,
								0x2C, 0x2D, 0x2E, 0x30, 0x31, 0x32, 0x34, 0x35,
								0x36, 0x38, 0x39, 0x3A, 0x3C, 0x3D, 0x3E };

// any block that is not 0x00 or in allBlocks but not in allowedBlocks
const BYTE dangerousSectorBlocks[] = { 0x07, 0x0B, 0x0F, 0x13, 0x17, 0x1B, 0x1F,
									   0x23, 0x27, 0x2B, 0x2F, 0x33, 0x37, 0x3B, 0x3F };

// for dumping entire card (read only)
const BYTE allBlocks[] = {  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 
						    0x08, 0x09, 0x0A, 0x0B, 0x0C, 0x0D, 0x0E, 0x0F,
							0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17,
							0x18, 0x19, 0x1A, 0x1B, 0x1C, 0x1D, 0x1E, 0x1F,
							0x20, 0x21, 0x22, 0x23, 0x24, 0x25, 0x26, 0x27,
							0x28, 0x29, 0x2A, 0x2B, 0x2C, 0x2D, 0x2E, 0x2F,
							0x30, 0x31, 0x32, 0x33, 0x34, 0x35, 0x36, 0x37,
							0x38, 0x39, 0x3A, 0x3B, 0x3C, 0x3D, 0x3E, 0x3F };

void PrintHex(LPCBYTE pbData, DWORD cbData)
{
	DWORD i;
	for (i = 0; i < cbData; i++)
	{
		wprintf(L"%02x ", pbData[i]);
	}
	wprintf(L"\n");
}

BOOL SendRecvReader(PSCARD_DUAL_HANDLE pHandle, const BYTE* pbData, const UINT16 cbData, BYTE* pbResult, UINT16* pcbResult)
{
	BOOL status = FALSE;
	DWORD cbRecvLenght = *pcbResult;
	LONG scStatus;

	wprintf(L"> ");
	PrintHex(pbData, cbData);

	scStatus = SCardTransmit(pHandle->hCard, NULL, pbData, cbData, NULL, pbResult, &cbRecvLenght);
	if (scStatus == SCARD_S_SUCCESS)
	{
		*pcbResult = (UINT16)cbRecvLenght;

		wprintf(L"< ");
		PrintHex(pbResult, *pcbResult);

		status = TRUE;
	}
	else wprintf(L"%08x\n", scStatus);

	return status;
}

BOOL OpenReader(LPCWSTR szReaderName, PSCARD_DUAL_HANDLE pHandle)
{
	BOOL status = FALSE;
	LONG scStatus;
	DWORD dwActiveProtocol;

	scStatus = SCardEstablishContext(SCARD_SCOPE_SYSTEM, NULL, NULL, &pHandle->hContext);
	if (scStatus == SCARD_S_SUCCESS)
	{
		scStatus = SCardConnect(pHandle->hContext, szReaderName, SCARD_SHARE_SHARED, SCARD_PROTOCOL_Tx, &pHandle->hCard, &dwActiveProtocol);
		if (scStatus == SCARD_S_SUCCESS)
		{
			status = TRUE;
		}
		else
		{
			SCardReleaseContext(pHandle->hContext);
		}
	}

	return status;
}

void CloseReader(PSCARD_DUAL_HANDLE pHandle)
{
	SCardDisconnect(pHandle->hCard, SCARD_LEAVE_CARD);
	SCardReleaseContext(pHandle->hContext);
}

int WriteToTag(const BYTE* Msg, BYTE block, bool allowUnsafeTargetBlocks) {
	bool allowedBlock = false;
	
	if (!allowUnsafeTargetBlocks) {
		for (int i = 0; i < sizeof(allowedBlocks); ++i) {
			if (allowedBlocks[i] == block) {
				allowedBlock = true;
				break;
			}
		}
	
		if (!allowedBlock) {
			wprintf(L"Target block is invalid.\n");
			return 1;
		}
	}
	// ignore safety and write to given block if true was passed for allowUnsafeTargetBlocks
	else {
		allowedBlock = true;
	}

	BYTE APDU_Write[5 + 16] = { 0xff, 0xd6, 0x00, block, 0x10 };	// base command 5 bytes + msg up to 16 bytes
	UINT16 apduWriteBaseLength = sizeof(APDU_Write) / sizeof(APDU_Write[0]);
	memcpy(APDU_Write + 5, Msg, 16);

	// use line below for DEFAULT KEY A
	const BYTE APDU_LoadDefaultKey[] = { 0xff, 0x82, 0x00, 0x00, 0x06, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff };

	const BYTE APDU_Authenticate_Block[] = { 0xff, 0x86, 0x00, 0x00, 0x05, 0x01, 0x00, block, 0x60, 0x00 };

	SCARD_DUAL_HANDLE hDual;
	BYTE Buffer[32];
	UINT16 cbBuffer;	// usually will be 2 (e.g. response 90 00 for success)

	// preparations are complete
	printf("Writing Hex:\n");
	for (UINT16 i = 0; i < 21; ++i) {		// first few chars are not part of message that will be written so skip printing them
		printf("0x%02X ", APDU_Write[i]);
	}
	printf("\n");


	// my Laptop:	"ACS ACR122U PICC Interface 0"
	// my PC:		"ACS ACR122 0"
	if (OpenReader(L"ACS ACR122 0", &hDual))
	{

		cbBuffer = 2;
		if (SendRecvReader(&hDual, APDU_LoadDefaultKey, sizeof(APDU_LoadDefaultKey), Buffer, &cbBuffer))
		{
			wprintf(L"Default Key A has been loaded.\n");
		}
		// make sure this operation was successful, terminate if not
		if (!(Buffer[0] == 0x90 && Buffer[1] == 0x00)) {
			CloseReader(&hDual);
			wprintf(L"Error code received. Aborting..\n");
			free(APDU_Write);
			return 1;
		}

		cbBuffer = 2;
		if (SendRecvReader(&hDual, APDU_Authenticate_Block, sizeof(APDU_Authenticate_Block), Buffer, &cbBuffer))
		{
			wprintf(L"Block has been authenticated.\n");
		}
		// make sure this operation was successful, terminate if not
		if (!(Buffer[0] == 0x90 && Buffer[1] == 0x00)) {
			CloseReader(&hDual);
			wprintf(L"Error code received. Aborting..\n");
			free(APDU_Write);
			return 1;
		}

		cbBuffer = 2;
		if (SendRecvReader(&hDual, APDU_Write, 23, Buffer, &cbBuffer))
		{
			wprintf(L"Data has successfully been written to the block!\n");
		}
		// make sure this operation was successful, terminate if not
		if (!(Buffer[0] == 0x90 && Buffer[1] == 0x00)) {
			CloseReader(&hDual);
			wprintf(L"Error code received. Aborting..\n");
			free(APDU_Write);
			return 1;
		}

		CloseReader(&hDual);
	}
	else {
		wprintf(L"Failed to find NFC reader.\n");
	}

	//free(APDU_Write);

	return 0;
}

int ReadFromTag(BYTE block, bool dump) {
	
	const BYTE APDU_Read[] = { 0xff, 0xb0, 0x00, block, 0x10 };	// page 16 of ACR122U_APIDriverManual.pdf (reads 16 bytes, and the page number is a coincidence)
	UINT16 apduReadLength = sizeof(APDU_Read) / sizeof(APDU_Read[0]);


	const BYTE APDU_LoadDefaultKey[] = { 0xff, 0x82, 0x00, 0x00, 0x06, 0xff, 0xff, 0xff, 0xff, 0xff, 0xff };
	const BYTE APDU_Authenticate_Block[] = { 0xff, 0x86, 0x00, 0x00, 0x05, 0x01, 0x00, block, 0x60, 0x00 };

	SCARD_DUAL_HANDLE hDual;
	BYTE Buffer[18];    // return status (success 90 00 or failure 63 00) takes 2 bytes, received data takes 16 bytes
	UINT16 cbBuffer;	// usually will be 2 (e.g. response 90 00 for success)


	// my Laptop:	"ACS ACR122U PICC Interface 0"
	// my PC:		"ACS ACR122 0"
	if (OpenReader(L"ACS ACR122 0", &hDual))
	{

		cbBuffer = 2;
		if (SendRecvReader(&hDual, APDU_LoadDefaultKey, sizeof(APDU_LoadDefaultKey), Buffer, &cbBuffer))
		{
			wprintf(L"Default Key A has been loaded.\n");
		}
		// make sure this operation was successful, terminate if not
		if (!(Buffer[0] == 0x90 && Buffer[1] == 0x00)) {
			CloseReader(&hDual);
			wprintf(L"Error code received. Aborting..\n");
			return 1;
		}

		cbBuffer = 2;
		if (SendRecvReader(&hDual, APDU_Authenticate_Block, sizeof(APDU_Authenticate_Block), Buffer, &cbBuffer))
		{
			wprintf(L"Block has been authenticated.\n");
		}
		// make sure this operation was successful, terminate if not
		if (!(Buffer[0] == 0x90 && Buffer[1] == 0x00)) {
			CloseReader(&hDual);
			wprintf(L"Error code received. Aborting..\n");
			return 1;
		}

		cbBuffer = 18;
		if (SendRecvReader(&hDual, APDU_Read, apduReadLength, Buffer, &cbBuffer))
		{
			// PrintHex(Buffer, sizeof(Buffer));
			// check for success (here the success code is stored after the data read, so read the last two bytes of the buffer)
			if (!(Buffer[16] == 0x90 && Buffer[17] == 0x00)) {
				CloseReader(&hDual);
				wprintf(L"Error code received. Aborting..\n");
				return 1;
			}
			
		}
		else {
			wprintf(L"Failed to read block.");
			CloseReader(&hDual);
			return 1;
		}

		// print actual characters in cmd if we are not dumping the full card
		if (!dump) {
			wprintf(L"Successfully read block.");
			wprintf(L"Trying to print read data in human-readable form:\n");
			for (int i = 0; i < 16; i++) {		// any text would have ended with \n so dont read last Byte
				printf("%c ", Buffer[i]);
			}
			printf("\n");
		}

		// if dump then write read data to fille
		if (dump) {
			// open file
			FILE* fp;
			int err = fopen_s(&fp, "dump.txt", "a");
			if (err != 0) {
				printf("Error opening the file.\n");
				CloseReader(&hDual);
				return 1;
			}

			if (fp != NULL) {
				// each line of the file should start with block #
				fprintf(fp, "[%02X] ", block);
				// write from BUFFER (but not last two status bytes)
				for (int i = 0; i < 16; i++) {
					fprintf(fp, "%02X ", Buffer[i]);
				}
				fprintf(fp, "\n"); // newline to prepare for next iteration

				// close file
				fclose(fp);
			}
			else {
				printf("ERROR: FD is zero. Stopping execution..\n");
				CloseReader(&hDual);
				return 1;
			}

		}


		CloseReader(&hDual);
	}
	else {
		wprintf(L"Failed to find NFC reader.\n");
	}

	return 0;
}

int ResetCardContents() {
	// empty every writable block
	const BYTE Msg[16] = { 0x00 };
	
	// get size of global allowedBlocks array (amount of elements)
	UINT16 arraySize = sizeof(allowedBlocks) / sizeof(allowedBlocks[0]);

	for (UINT16 i = 0; i < arraySize; ++i) {
		UINT16 status_code = WriteToTag(Msg, allowedBlocks[i], false);
		if (status_code != 0) {
			printf("Error occured while writing! Terminating.");
			return 1;
		}
	}

	printf("\nSuccessfully reset card contents.\n");
	return 0;
}

int DumpCard() {
	// get size of global allowedBlocks array (amount of elements)
	UINT16 arraySize = sizeof(allBlocks) / sizeof(allBlocks[0]);

	for (UINT16 i = 0; i < arraySize; ++i) {
		UINT16 status_code = ReadFromTag(allBlocks[i], true);
		if (status_code != 0) {
			printf("Error occured while trying to dump card! Terminating.");
			return 1;
		}
		// wait between iterations to avoid errors with opening/closing file
		Sleep(100);	// 100 ms usually works, shorter times can lead to a failed dump
	}

	printf("\nSuccessfully dumped card content to dump.txt.\n");
	return 0;
}

// formats the card for NDEF (only works once because it assumes key A of every sector to be the default one
// after running this code the mifare classic 1k becomes compatible with almost every modern smartphone!! :)
int FormatNDEF() {
	// STEP 1: Adjust block 1
	const BYTE block1msg[16] = { 0x14, 0x01, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1 };
	BYTE block1Block = 0x01;
	int step1Success = WriteToTag(block1msg, block1Block, false);
	if (step1Success != 0) {
		printf("Error occurred while writing to block 1 of sector 0! Terminating.");
		return 1;
	}
	Sleep(100);
	
	// STEP 2: Adjust block 2
	const BYTE block2msg[16] = { 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1 };
	BYTE block2Block = 0x02;
	int step2Success = WriteToTag(block2msg, block2Block, false);
	if (step2Success != 0) {
		printf("Error occurred while writing to block 2 of sector 0! Terminating.");
		return 1;
	}
	Sleep(100);

	// STEP 3: Adjust trailer sector 0
	const BYTE SectorZeroMsg[16] = { 0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5, 0x78, 0x77, 0x88, 0xC1, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF };
	// const BYTE SectorZeroMsg[16] = { 0xA0, 0xA1, 0xA2, 0xA3, 0xA4, 0xA5, 0x78, 0x77, 0x88, 0xC1, 0x89, 0xEC, 0xA9, 0x7F, 0x8C, 0x2A };
	BYTE SectorZeroBlock = 0x03;
	int step3Success = WriteToTag(SectorZeroMsg, SectorZeroBlock, true);
	if (step3Success != 0) {
		printf("Error occurred while writing trailer sector 0! Terminating.");
		return 1;
	}
	Sleep(100);

	// STEP 4: Write empty NDEF message to block 0 of sector 1
	const BYTE NDEFmsg[16] = { 0x03, 0x00, 0xFE, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00 };
	BYTE NDEFmsgBlock = 0x04;
	int step4Success = WriteToTag(NDEFmsg, NDEFmsgBlock, false);
	if (step4Success != 0) {
		printf("Error occurred while writing to block 0 of sector 1! Terminating.");
		return 1;
	}
	Sleep(100);

	// STEP 5: Adjust trailer sectors of sectors 1-15
	const BYTE Msg[16] = { 0xD3, 0xF7, 0xD3, 0xF7, 0xD3, 0xF7, 0x7F, 0x07, 0x88, 0x40, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF, 0xFF };
	// get size of global dangerousSectorBlocks array (amount of elements)
	UINT16 arraySize = sizeof(dangerousSectorBlocks) / sizeof(dangerousSectorBlocks[0]);
	for (UINT16 i = 0; i < arraySize; ++i) {
		UINT16 status_code = WriteToTag(Msg, dangerousSectorBlocks[i], true);
		if (status_code != 0) {
			printf("Error occurred while writing! Terminating.");
			return 1;
		}
		// wait between iterations to be safe (might not be required tho)
		Sleep(100);
	}

	printf("\nSuccessfully NDEF formatted the card.\n");
	return 0;

}

int main() {
	//const BYTE Msg[16] = { 0x00 };	// empty the given block
	
	// block 01
	//const BYTE Msg[16] = { 0x14, 0x01, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1 };
	//BYTE block = 0x01;
	
	// block 02
	//const BYTE Msg[16] = { 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1, 0x03, 0xE1 };
	//BYTE block = 0x02;

	// ----------------------
	// determine size of array now (because you cant do that later on from a pointer)
	//UINT16 msgSize = sizeof(Msg) / sizeof(Msg[0]);
	// ----------------------

	// choose function to run (uncomment):
	// WriteToTag(Msg, block, false);
	// ReadFromTag(block, false);	// true / false (store block content in file dump.txt / dont)
	// ResetCardContents();	// only works with default keys / sector trailers
	// DumpCard();
	FormatNDEF();

	return 0;
}

```

## Conclusion
I learned a lot about the ACR122U reader, Mifare Classic tags and APDU! I managed to write data to my Mifare Classic 1k tag and to NDEF-format it so that it becomes compatible with mobile devices! In the future, I might write functions that split up long input strings automatically across valid blocks etc. Ideas are endless, only time is short :). If you are interested in NFC topics I recommend checking out the Iceman discord server, it has the brightest minds of RFID and is very helpful and active.

