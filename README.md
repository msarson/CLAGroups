# CLAGroups

## Helper for unmanaged exports use with clarion

Nuget package can be found [here](http://https://www.nuget.org/packages/ClaGroups "here") 

The packages is a .Net Standard 2.0 project and supports the following versions of .Net

.Net Core 2.0 and above
.Net Framework 4.6.2 and above
See notes [here](https://docs.microsoft.com/en-us/dotnet/standard/net-standard "here")

To use the package in visual studio (I have only tested in 2019) Search for the package CLAGroups

More documentation is coming to show how to use the package.

I am also working on a clarion class that will create the needed structure and class for using your group with .net.

Simple usage
```clarion
!Clarion group that transposes to the C# code below
Person                      GROUP
Name                            CSTRING(100)
Age                             LONG
Message                         STRING(10)
                            END
```                            
```csharp
//Code to start working with a group
//This code will automatically handle the IntPtr passed as ADDRESS(Group)
//From clarion to the c# structure and class shown below
//ptrToPerson is defined as and will usually be part of an exported method

[DllExport("PassGroup", CallingConvention = CallingConvention.StdCall)]
public static void PassGroup(IntPtr ptrToPerson)
{
  //Constructor will automagically do the magic pointer stuff
  PersonDetails person = new PersonDetails(ptrToPerson);
  //Do your magic then save back to memory
  person.UpdateGroupMemory();
}

//The following will be the code needed to do the rest
//This code will be generated by the clarion class I am creating and will
//need to be part of whatever project you are working on.


//Structure definition of the clarion group
[StructLayout(LayoutKind.Sequential, Pack = 1)]
public struct Person
{
    [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 100)]
    public string Name;             //Clarion CSTRING definition
    public int Age;                 //Clarion LONG definition
    [MarshalAs(UnmanagedType.ByValArray, SizeConst = 10)]
    public byte[] Message;          //Clarion STRING definition
}

//Class definition that will manage the group
public class PersonDetails : GroupBaseClass<Person>
{
    public PersonDetails(IntPtr ptr) : base(ptr)
    {
    }
    public int Age
    {
        get => group.Age;
        set
        {
            group.Age = value;
            OnPropertyChanged("Age");
        }
    }
    public string Message
    {
        get
        {
            return Encoding.UTF8.GetString((byte[])group.Message, 0, group.Message.Length);
        }
        set
        {
            group.Message = Encoding.UTF8.GetBytes(value);
            OnPropertyChanged("Message");
        }
    }
    public string Name
    {
        get => group.Name;
        set
        {
            group.Name = value;
            OnPropertyChanged("Name");
        }
    }
}
