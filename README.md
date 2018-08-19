# Tutorial 1 - Strings Encryption

##Intro.
String encryption is a protection method to convert Readable string to Unreadable String.

![image](https://i.imgur.com/SIU5V30.png)

##Let's Start.
First of all we have to Select the application that we want to Obfuscate it.

```csharp
ModuleDef md = ModuleDefMD.Load(@"AppPath"); //replace AppPath with your Application you want to Obfuscate it.
```
Then we have to load the Types from the Module.
```csharp
foreach(TypeDef type in md.Types)
{

}
```
after we loaded the Types from module, now we have to get the methods from the type.
```csharp
foreach(MethodDef method in type.Methods)
{

}
```
After we loaded the Methods from type, now we need to get the Instructions for method and check if there's any String (String OpCode = Ldstr). But we should check if the method body isn't Null.
```csharp
if (method.Body == null) continue;
```
Now we are going to get the instructions from method.
```csharp
for(int i = 0; i < method.Body.Instructions.Count; i++)
{
                        
}
```
after getting the instructions from method, and check it if its null or no. now we have to check if the instruction is String
```csharp
if (method.Body.Instructions[i].OpCode == OpCodes.Ldstr)
{
}
```
I will use Base64 for Encrypt the String and UTF8 for Encoding.
```csharp
String oldString = method.Body.Instructions[i].Operand.ToString(); //Original String.
String newString = Convert.ToBase64String(UTF8Encoding.UTF8.GetBytes(oldString)); //Encrypted String by Base64
```
Now we have to Inject the Decrypt Instructions

###Decrypt Instructions

![image](https://i.imgur.com/t3gGKzl.png)

To Dnlib Inject
```csharp
method.Body.Instructions[i].OpCode = OpCodes.Nop; //Change the Opcode for the Original Instruction
method.Body.Instructions.Insert(i + 1, new Instruction(OpCodes.Call, md.Import(typeof(System.Text.Encoding).GetMethod("get_UTF8", new Type[] { })))); //get Method (get_UTF8) from Type (System.Text.Encoding).
method.Body.Instructions.Insert(i + 2, new Instruction(OpCodes.Ldstr, newString)); //add the Encrypted String
method.Body.Instructions.Insert(i + 3, new Instruction(OpCodes.Call, md.Import(typeof(System.Convert).GetMethod("FromBase64String", new Type[] { typeof(string) })))); //get Method (FromBase64String) from Type (System.Convert), and arguments for method we will get it using "new Type[] { typeof(string) }"
method.Body.Instructions.Insert(i + 4, new Instruction(OpCodes.Callvirt, md.Import(typeof(System.Text.Encoding).GetMethod("GetString", new Type[] { typeof(byte[]) }))));
i += 4; //skip the Instructions that we have just added.
```
Now we're almost done, now we only need to Write the obfuscated module.
```csharp
md.Write("ObfuscatedApp.exe");
```
Done, Enjoy :)).

###the code should be like this
```csharp
ModuleDef md = ModuleDefMD.Load(@"AppPath");
foreach(TypeDef type in md.Types)
{
    foreach (MethodDef method in type.Methods)
    {
    if (method.Body == null) continue;
        for(int i = 0; i < method.Body.Instructions.Count(); i++)
        {
            if (method.Body.Instructions[i].OpCode == OpCodes.Ldstr)
            {
                //Encoding.UTF8.GetString(Convert.FromBase64String(""));
                String oldString = method.Body.Instructions[i].Operand.ToString(); //Original String.
                String newString = Convert.ToBase64String(UTF8Encoding.UTF8.GetBytes(oldString)); //Encrypted String by Base64
                method.Body.Instructions[i].OpCode = OpCodes.Nop; //Change the Opcode for the Original Instruction
                method.Body.Instructions.Insert(i + 1, new Instruction(OpCodes.Call, md.Import(typeof(System.Text.Encoding).GetMethod("get_UTF8", new Type[] { })))); //get Method (get_UTF8) from Type (System.Text.Encoding).
                method.Body.Instructions.Insert(i + 2, new Instruction(OpCodes.Ldstr, newString)); //add the Encrypted String
                method.Body.Instructions.Insert(i + 3, new Instruction(OpCodes.Call, md.Import(typeof(System.Convert).GetMethod("FromBase64String", new Type[] { typeof(string) })))); //get Method (FromBase64String) from Type (System.Convert), and arguments for method we will get it using "new Type[] { typeof(string) }"
                method.Body.Instructions.Insert(i + 4, new Instruction(OpCodes.Callvirt, md.Import(typeof(System.Text.Encoding).GetMethod("GetString", new Type[] { typeof(byte[]) }))));
                i += 4; //skip the Instructions that we have just added.
            }
        }
    }
}
md.Write("ObfuscatedApp.exe");
```
