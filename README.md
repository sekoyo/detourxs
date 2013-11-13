DetourXS
========

You can use DetourXS to intercept functions at assembly level for Windows OS. I believe it is the only one that supports detour chaining, which is useful for plugin architecture amongst other things (though as of present limitations apply to keep the code simple).

Features
--------

- x86/64 compatible without changing code
- Uses a lightweight length dasm
- Allows for infinite detour chaining
- Uses absolute detours only where necessary (i.e. 64bit jumps)
- Easy to use, easy to set up

How to include
--------------

Copy the “DetourXS” folder into your source folder and include, #include “DetourXS/detourxs.h”

Example
-------

	#include <Windows.h>
	#include <iostream>
	#include "DetourXS/detourxs.h"

	typedef DWORD (WINAPI* tGetTickCount)();
	tGetTickCount GetTickCountOrigFirst = NULL;
	tGetTickCount GetTickCountOrigSecond = NULL;
	tGetTickCount GetTickCountOrigThird = NULL;

	DWORD WINAPI GetTickCountHookFirst()
	{
		std::cout << "GetTickCount FIRST HOOK called\n";
		return GetTickCountOrigFirst();
	}

	DWORD WINAPI GetTickCountHookSecond()
	{
		std::cout << "GetTickCount SECOND HOOK called\n";
		return GetTickCountOrigSecond();
	}

	DWORD WINAPI GetTickCountHookThird()
	{
		std::cout << "GetTickCount THIRD HOOK called\n";
		return GetTickCountOrigThird();
	}


	int main()
	{
		DetourXS gtcFirst(&GetTickCount, GetTickCountHookFirst);
		GetTickCountOrigFirst = (tGetTickCount) gtcFirst.GetTrampoline();

		DetourXS gtcSecond(&GetTickCount, GetTickCountHookSecond);
		GetTickCountOrigSecond = (tGetTickCount) gtcSecond.GetTrampoline();

		DetourXS gtcThird(&GetTickCount, GetTickCountHookThird);
		GetTickCountOrigThird = (tGetTickCount) gtcThird.GetTrampoline();

		std::cout << "Calling GetTickCount\n";
		GetTickCount();

		// Presently in order to not break the chain you can only remove the detours in this order
		// - It won't crash if you don't but you will break the chain
		gtcThird.Destroy();
		gtcSecond.Destroy();

		std::cout << "Calling GetTickCount\n";
		GetTickCount();

		return 0;
	}

Limitations
-----------

- Fixing of relative opcodes in the trampoline (until this is done some function hooking will cause crashes)
- Maybe class detouring

Credits
-------

LDE64 engine (http://www.beaengine.org/)

Licence
-------
Type: wtfpl
Link: http://www.wtfpl.net/

Notes
-----

Possible crashes: The detour currently doesn’t fix relative opcodes in the trampolene. Therefore relative opcodes can jump/call to the wrong places, or read/write to wrong places.

Chaining: There is currently an issue with chaining: If a detour is removed that isn’t the last one in the chain this will break the chain for the ones after it. I think solutions for this could become very messy but if anyone has ideas please let me know. I thought about when a detour is Destroy()ed to fix its own patched bytes if it has any, and then repointing what it terms as the original (m_lpFuncOrig) to the address the patch was pointing at (aka the one next in the chain). This would work but then how could you destroy the chain afterwards since you just destroyed the class capable of restoring the m_lpFuncOrig bytes!?