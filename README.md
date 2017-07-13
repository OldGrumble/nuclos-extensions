# nuclos-extensions

==================================== File: NuclosLayoutDataObject =================================================

/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package de.epdv.nuclos.editor.layout.data;

import java.io.IOException;
import java.util.Enumeration;
import java.util.HashSet;
import java.util.Set;
import org.netbeans.core.spi.multiview.MultiViewElement;
import org.netbeans.core.spi.multiview.text.MultiViewEditorElement;
import org.openide.filesystems.FileObject;
import org.openide.filesystems.FileSystem;
import org.openide.filesystems.MIMEResolver;
import org.openide.loaders.DataObject;
import org.openide.loaders.DataObjectExistsException;
import org.openide.loaders.MultiDataObject;
import org.openide.loaders.MultiFileLoader;
import org.openide.nodes.CookieSet;
import org.openide.nodes.Node;
import org.openide.text.DataEditorSupport;
import org.openide.util.Lookup;
import org.openide.util.NbBundle.Messages;
import org.openide.util.lookup.AbstractLookup;
import org.openide.util.lookup.InstanceContent;
import org.openide.util.lookup.ProxyLookup;
import org.openide.windows.TopComponent;

//@Messages({
//    "LBL_NuclosLayout_LOADER=Files of NuclosLayout"
//})
//@MIMEResolver.ExtensionRegistration(
//        displayName = "#LBL_NuclosLayout_LOADER",
//        mimeType = "text/x-nuclos-layoutml+xml",
//        extension = {"layoutml"},
//        showInFileChooser = "Nuclos Layout Files"
//)
@DataObject.Registration(
        mimeType = "text/x-nuclos-layoutml+xml",
        iconBase = "de/epdv/nuclos/editor/layout/resources/nuclos-layout.png",
        displayName = "NuclosLayout Files",
        position = 300
)
//@ActionReferences({
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "System", id = "org.openide.actions.OpenAction"),
//            position = 100,
//            separatorAfter = 200
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "Edit", id = "org.openide.actions.CutAction"),
//            position = 300
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "Edit", id = "org.openide.actions.CopyAction"),
//            position = 400,
//            separatorAfter = 500
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "Edit", id = "org.openide.actions.DeleteAction"),
//            position = 600
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "System", id = "org.openide.actions.RenameAction"),
//            position = 700,
//            separatorAfter = 800
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "System", id = "org.openide.actions.SaveAsTemplateAction"),
//            position = 900,
//            separatorAfter = 1000
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "System", id = "org.openide.actions.FileSystemAction"),
//            position = 1100,
//            separatorAfter = 1200
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "System", id = "org.openide.actions.ToolsAction"),
//            position = 1300
//    ),
//    @ActionReference(
//            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
//            id = @ActionID(category = "System", id = "org.openide.actions.PropertiesAction"),
//            position = 1400
//    )
//})
public class NuclosLayoutDataObject extends MultiDataObject {

    private Lookup lookup;

    @SuppressWarnings("LeakingThisInConstructor")
    public NuclosLayoutDataObject(FileObject primaryFile, MultiFileLoader loader) throws DataObjectExistsException, IOException {
        super(primaryFile, loader);
//        CookieSet cookies = getCookieSet();
//        cookies.add((Node.Cookie) DataEditorSupport.create(this, getPrimaryEntry(), cookies));
        registerEditor("text/x-nuclos-layoutml+xml", true);
        for (FileObject file : files()) {
            if (file != primaryFile) {
                ((NuclosLayoutFileLoader)loader).createSecondaryEntry(this, file);
            }
        }
    }

    @Override
    protected int associateLookup() {
        return 1;
    }

    @Override
    public Lookup getLookup() {
        if (lookup == null) {
            NuclosLayoutFilesProvider nlfp = new NuclosLayoutFilesProvider(files());
            InstanceContent ic = new InstanceContent();
            ic.add(nlfp);
            lookup = new ProxyLookup(super.getLookup(), new AbstractLookup(ic));
        }
        return lookup;
    }

    @MultiViewElement.Registration(
            displayName = "#LBL_NuclosLayout_EDITOR",
            iconBase = "de/epdv/nuclos/editor/layout/resources/nuclos-layout.png",
            mimeType = "text/x-nuclos-layoutml+xml",
            persistenceType = TopComponent.PERSISTENCE_ONLY_OPENED,
            preferredID = "NuclosLayout",
            position = 1000
    )
    @Messages("LBL_NuclosLayout_EDITOR=Source")
    public static MultiViewEditorElement createEditor(Lookup lkp) {
        return new MultiViewEditorElement(lkp);
    }

    @Override
    public Set<FileObject> files() {
        Set<FileObject> files = new HashSet<>(4);
        FileObject fo = getPrimaryFile();
        String name = fo.getName();
        FileObject parent = fo.getParent();
        files.add(fo);
        files.add(parent.getFileObject(name + NuclosLayoutFileLoader.EXT_LAYOUT_EOML));
        FileObject subDir = parent.getFileObject(NuclosLayoutFileLoader.FOLDER_LAYOUT_USAGE);
        files.add(subDir);
        Enumeration<? extends FileObject> children = subDir.getChildren(false);
        while (children.hasMoreElements()) {
            FileObject child = children.nextElement();
            if (child.isData() && child.getNameExt().endsWith(NuclosLayoutFileLoader.EXT_USAGE_EOML)) {
                files.add(child);
            }
        }
        return files;
    }
}



================================================= File: NuclosLayoutFileLoader ======================================


/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package de.epdv.nuclos.editor.layout.data;

import java.io.IOException;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;
import org.openide.awt.ActionID;
import org.openide.awt.ActionReference;
import org.openide.awt.ActionReferences;
import org.openide.filesystems.FileObject;
import org.openide.filesystems.MIMEResolver;
import org.openide.loaders.DataObject;
import org.openide.loaders.DataObjectExistsException;
import org.openide.loaders.FileEntry;
import org.openide.loaders.MultiDataObject;
import org.openide.loaders.MultiFileLoader;
import org.openide.util.NbBundle;

/**
 *
 * @author peter
 */
@NbBundle.Messages({
    "LBL_NuclosLayout_LOADER=Files of NuclosLayout"
})
@MIMEResolver.ExtensionRegistration(
        displayName = "#LBL_NuclosLayout_LOADER",
        mimeType = "text/x-nuclos-layoutml+xml",
        extension = {"layoutml"},
        showInFileChooser = "Nuclos Layout Files"
)
@DataObject.Registration(
        mimeType = "text/x-nuclos-layoutml+xml",
        iconBase = "de/epdv/nuclos/editor/layout/resources/nuclos-layout.png",
        displayName = "#LBL_NuclosLayout_LOADER",
        position = 300
)
@ActionReferences({
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "System", id = "org.openide.actions.OpenAction"),
            position = 100,
            separatorAfter = 200
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "Edit", id = "org.openide.actions.CutAction"),
            position = 300
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "Edit", id = "org.openide.actions.CopyAction"),
            position = 400,
            separatorAfter = 500
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "Edit", id = "org.openide.actions.DeleteAction"),
            position = 600
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "System", id = "org.openide.actions.RenameAction"),
            position = 700,
            separatorAfter = 800
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "System", id = "org.openide.actions.SaveAsTemplateAction"),
            position = 900,
            separatorAfter = 1000
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "System", id = "org.openide.actions.FileSystemAction"),
            position = 1100,
            separatorAfter = 1200
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "System", id = "org.openide.actions.ToolsAction"),
            position = 1300
    ),
    @ActionReference(
            path = "Loaders/text/x-nuclos-layoutml+xml/Actions",
            id = @ActionID(category = "System", id = "org.openide.actions.PropertiesAction"),
            position = 1400
    )
})
public class NuclosLayoutFileLoader extends MultiFileLoader {

    static final String EXT_LAYOUT_NO_DOT = "layoutml";
    static final String EXT_LAYOUT_EOML = ".nuclos_layout.eoml";
    static final String FOLDER_LAYOUT_USAGE = "nuclos_layoutusage";
    static final String EXT_USAGE_EOML = ".nuclos_layoutusage.eoml";

    public NuclosLayoutFileLoader() {
        super("de.epdv.nuclos.editor.layout.data.NuclosLayoutDataObject");
    }

    @Override
    protected FileObject findPrimaryFile(FileObject fo) {
        if (fo.getExt().equals(EXT_LAYOUT_NO_DOT)) {
            return fo;
        } else {
            String name = fo.getNameExt();
            if (name.endsWith(EXT_LAYOUT_EOML)) {
                String baseName = name.substring(0);
                return fo.getParent().getFileObject(baseName + '.' + EXT_LAYOUT_NO_DOT);
            } else if (name.endsWith(EXT_USAGE_EOML) && fo.getParent().getNameExt().equals(FOLDER_LAYOUT_USAGE)) {
                return getLayoutChild(fo.getParent().getParent());
            } else if (fo.isFolder() && fo.getNameExt().equals(FOLDER_LAYOUT_USAGE)) {
                return getLayoutChild(fo.getParent());
            }
        }
        return null;
    }

    @Override
    protected MultiDataObject createMultiObject(FileObject fo) throws DataObjectExistsException, IOException {
        return new NuclosLayoutDataObject(fo, this);
    }

    @Override
    protected MultiDataObject.Entry createPrimaryEntry(MultiDataObject mdo, FileObject fo) {
        return new FileEntry(mdo, fo);
    }

    @Override
    protected MultiDataObject.Entry createSecondaryEntry(MultiDataObject mdo, FileObject fo) {
        MultiDataObject.Entry entry;
        if (fo.isFolder()) {
            entry = new FileEntry.Folder(mdo, fo);
        } else {
            entry = new FileEntry(mdo, fo);
        }
        return entry;
    }

    private FileObject getLayoutChild(FileObject fo) {
        Enumeration<? extends FileObject> children = fo.getChildren(false);
        List<FileObject> listChildren = new ArrayList<>();
        while (children.hasMoreElements()) {
            FileObject child = children.nextElement();
            if (child.getExt().equals(EXT_LAYOUT_NO_DOT)) {
                listChildren.add(child);
            }
        }
        return (listChildren.size() == 1) ? listChildren.get(0) : null;
    }
}
