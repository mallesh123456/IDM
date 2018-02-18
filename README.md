# IDM
Oimuseroperations
///*
//* 
// *Class Description - This scheduler can be used in case of granting a Role to bulk users. 
// *                    This requires one csv file in which User Logins will be there to which the role to be granted. 
// *                    The csv’s User Logins should be separated by comma. For example: UserLogin1,UserLogin2,UserLogin3…
//*
//*Description Ended
//**/
package com.ril;

import Thor.API.Exceptions.tcAPIException;
import Thor.API.Exceptions.tcColumnNotFoundException;
import Thor.API.Operations.tcGroupOperationsIntf;
import Thor.API.tcResultSet;

import java.io.File;
import java.io.FileNotFoundException;

import java.util.HashMap;
import java.util.HashSet;
import java.util.Map;
import java.util.Scanner;
import java.util.Set;

import oracle.iam.identity.exception.NoSuchUserException;
import oracle.iam.identity.exception.SearchKeyNotUniqueException;
import oracle.iam.identity.exception.UserLookupException;
import oracle.iam.identity.rolemgmt.api.RoleManager;
import oracle.iam.identity.usermgmt.api.UserManager;
import oracle.iam.identity.usermgmt.vo.User;
import oracle.iam.platform.OIMClient;
import oracle.iam.platform.Platform;
import oracle.iam.platform.kernel.ValidationFailedException;




public class RoleAssign {
    String className="RoleAssignment";
    static OIMClient provOimClient;
    static Integer uscount;
    static String tempretval;
    public RoleAssign() {
        super();
    }

    public static void main(String args[]) {
        try {
        	Oimconnection conn=new Oimconnection();
            provOimClient = conn.Oimconnection();
        RoleManager rmgr = null;
        rmgr = provOimClient.getService(RoleManager.class);
        uscount = 0;
        String csvPath = "C:\\Oimapi\\RoleAssignmentLogins.txt";  //C:\Oimapi
        String userLogins = getCsv(csvPath);
            
        String roleKey = getGroupKey("RRBEAT");// RPOSUser,PRMMaintainDealerUsers, In10s, Airwatch2,CCUserRole
            
        Set<String> userKeys = null;
        userKeys = new HashSet<String>();
        String tempUserKey = null;
        tempretval = "end";
            for (String retval : userLogins.split(",")) {
                tempUserKey = getUserKey(retval);
                userKeys.add(tempUserKey);
                if(!rmgr.isRoleGranted(roleKey, tempUserKey, true))
                {
                    System.out.println("Granting role to UserLogin = "+retval+" and Role key is = "+roleKey);
                    rmgr.grantRole(roleKey, userKeys);
                }
                else {
                    System.out.println("Role is already granted to UserLogin = "+retval);
                }
                userKeys.remove(tempUserKey);
                uscount++;
                tempretval = retval;
            }
        System.out.println("Success");
        } catch (ValidationFailedException e) {
            System.out.println("Exception Caught with message :"+e.getMessage()+". The number of records assigned is "+uscount);
            e.printStackTrace();
        } catch (Exception e) {
            System.out.println("Exception Caught with message :"+e.getMessage()+". The number of records assigned is "+uscount);
            e.printStackTrace();
        }
        finally{
            System.out.println("Record successfully executed till userLogin : "+tempretval);
            provOimClient.logout();
        }
    }

    public static String getCsv(String filePath) {
        
        Scanner scanner = null;
        String returnUserList = "";
       
        try {
            scanner = new Scanner(new File(filePath));
            scanner.useDelimiter("|");
           
            while (scanner.hasNext()) {
                returnUserList = returnUserList.concat(scanner.next());
            }
            System.out.println("Return User List is : "+returnUserList);
           
            scanner.close();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }
        return returnUserList;
    }

    public static String getGroupKey(final String groupName) {
        tcGroupOperationsIntf groupManager =
            provOimClient.getService(tcGroupOperationsIntf.class);
        Map condition = new HashMap();
        condition.put("Groups.Group Name", groupName);
        tcResultSet rs;
        String groupKey = null;
        try {
            rs = groupManager.findGroups(condition);
            groupKey = rs.getStringValue("Groups.Key");
        } catch (tcAPIException e) {
            e.printStackTrace();
        } catch (tcColumnNotFoundException e) {
            e.printStackTrace();
        }
        return groupKey;
    }

    public static String getUserKey(final String userLogin) {
        UserManager userManager = null;
        String userKey = null;
        try {
            if (userLogin == null) {
                System.out.println("ClientOIM ::  getUserKey :: UserLogin is either null or empty");
            } else {
                final Set<String> returnMap = new HashSet<String>();

                /* Initialize the User Manager Service */
                userManager = provOimClient.getService(UserManager.class);

                /* Check whether User Manager Service properly initialized */
                if (userManager != null) {
                    /* Get User object belonging to the User Key */
                    final User user =
                        userManager.getDetails("User Login", userLogin,
                                               returnMap);
                    /* If User object is not null then retrieve User Login */
                    if (user != null) {
                        userKey = user.getEntityId();
                    }
                }
            }

        } catch (NoSuchUserException e) {
            System.out.println("ClientOIM ::  getUserKey :: NoSuchUserException :: " +
                               e.getMessage());
        } catch (UserLookupException e) {
            System.out.println("ClientOIM ::  getUserKey :: UserLookupException :: " +
                               e.getMessage());
        } catch (SearchKeyNotUniqueException e) {
            System.out.println("ClientOIM ::  getUserKey :: SearchKeyNotUniqueException :: " +
                               e.getMessage());
        } catch (Exception e) {
            System.out.println("ClientOIM ::  getUserKey :: Exception :: " +
                               e.getMessage());
        }
        return userKey;
    }

}




****************************************
package com.ril;

import java.util.Hashtable;

import javax.security.auth.login.LoginException;

import oracle.iam.platform.OIMClient;

public class Oimconnection {
	public static OIMClient oimClient;
	
	public static OIMClient  Oimconnection()
	{
		Hashtable<Object, Object> env = new Hashtable<Object, Object>();
		env.put(OIMClient.JAVA_NAMING_FACTORY_INITIAL, "weblogic.jndi.WLInitialContextFactory");
		env.put(OIMClient.JAVA_NAMING_PROVIDER_URL, "t3://pdoim01.bss.jio.com:14000");
        System.setProperty("java.security.auth.login.config", "C://Softwares//designconsole//config//authwl.conf");
        System.setProperty("OIM.AppServerType", "wls");
        System.setProperty("APPSERVER_TYPE", "wls");
        oracle.iam.platform.OIMClient oimClient = new oracle.iam.platform.OIMClient(env);

		try {
        oimClient.login("xelsysadm", "JioIAM0315".toCharArray());
        System.out.println();
		System.out.print("Successfully Connected with OIM ");
		} catch (LoginException e) {
		System.out.print("Login Exception"+ e);
		
		}
		return  oimClient;
	}

	
	

}
