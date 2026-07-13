# Composite Pattern
- [What is the Composite Pattern](#what-is-the-composite-pattern)
- [The File System Analogy](#the-file-system-analogy)
- [Code Example (C#)](#code-example-c)
- [Sources](#sources)

## What is the Composite Pattern

The **Composite Pattern** is a structural design pattern that lets you compose objects into tree structures to represent part-whole hierarchies.
It allows clients to treat individual objects (leaf nodes) and groups of objects (composite nodes) uniformly.

Instead of writing code that needs to distinguish between a single item and a container, you define a common interface or base class that both implement.

## The File System Analogy

Imagine a computer **File System**:
- You have **Files** (Leaf). A file has a name and a size, but it cannot contain other items.
- You have **Folders** (Composite). A folder has a name and can contain files or other folders.

If you want to know the size of a folder, you sum the sizes of all its contents recursively.
If you want to know the size of a file, you just read its size.

By treating both files and folders under a single "file system item" abstraction, you can query the size of any item—whether it is a single file or a huge folder hierarchy—using the exact same method.

## Code Example (C#)

```csharp
using System;
using System.Collections.Generic;

// Component
public abstract class FileSystemItem {
    public string Name { get; }
    protected FileSystemItem(string name) {
        Name = name;
    }
    public abstract int GetSize();
    public abstract void Display(int indent);
}

// Leaf
public class FileItem : FileSystemItem {
    private readonly int _size;
    public FileItem(string name, int size) : base(name) {
        _size = size;
    }
    public override int GetSize() => _size;
    public override void Display(int indent) {
        Console.WriteLine($"{new string(' ', indent)}- {Name} ({_size} KB)");
    }
}

// Composite
public class FolderItem : FileSystemItem {
    private readonly List<FileSystemItem> _children = new();
    public FolderItem(string name) : base(name) {}

    public void Add(FileSystemItem item) {
        _children.Add(item);
    }

    public void Remove(FileSystemItem item) {
        _children.Remove(item);
    }

    public override int GetSize() {
        int totalSize = 0;
        foreach (var child in _children) {
            totalSize += child.GetSize();
        }
        return totalSize;
    }

    public override void Display(int indent) {
        Console.WriteLine($"{new string(' ', indent)}+ {Name}/ ({GetSize()} KB)");
        foreach (var child in _children) {
            child.Display(indent + 2);
        }
    }
}

public class Program {
    public static void Main() {
        // Individual files (Leaves)
        var file1 = new FileItem("resume.pdf", 120);
        var file2 = new FileItem("photo.png", 800);
        var file3 = new FileItem("notes.txt", 15);

        // Folders (Composites)
        var root = new FolderItem("Root");
        var docs = new FolderItem("Documents");
        var pics = new FolderItem("Pictures");

        // Build the tree structure
        docs.Add(file1);
        docs.Add(file3);
        pics.Add(file2);

        root.Add(docs);
        root.Add(pics);

        // Print folder structure and sizes
        Console.WriteLine("File System Structure:");
        root.Display(0);

        Console.WriteLine($"\nTotal Root Size: {root.GetSize()} KB");
    }
}
```

## Sources

- [Refactoring Guru: Composite Pattern](https://refactoring.guru/design-patterns/composite)
- [Composite Pattern (Wikipedia)](https://en.wikipedia.org/wiki/Composite_pattern)
