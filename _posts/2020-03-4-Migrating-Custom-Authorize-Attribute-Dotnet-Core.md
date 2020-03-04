---
layout: post
title: Migrating your Custom Authorize Attribute to .Net Core
---

## Why did we needed to do this?

Microsoft has made it clear that they prefer developers use policy based authentication as documented [here](https://docs.microsoft.com/en-us/aspnet/core/security/authorization/policies?view=aspnetcore-3.1).

At the beginning of February we decieded to seriously start our conversion from .Net 4.7.2 to .Net Core.  We were excited about the possibilities it will afford us from a hosting stand point as well as getting us onto the newer supported technologies.

One of the challenges tasked to me was to discover how we can utilize our existing Claims Authentication logic in .NET Core.  Because we have a lot of applications that need to convert our phase 1 approach is to make the changes as minimal as possible.  Then we can later start to leverage some of the framework capabilities to enhance our stack.

Our current logic works something like this.  You will have User Roles that have Claim Types and Claim Values associated to them.  A Claim Type may be `Customers` or `Orders` and a Claim Value may be `ReadOnly` or `FullAccess`.  We store the association of these claims to the roles in the DB and they can be updated dynamically.

Due to the dynamic nature, granularity of our claims and the fact that we want application upgrades to be as simple as possible we didn't want to utilize the policy based logic and wanted to try and mimic the process we had currently.

## How it works in .Net Framwork

We implement a custom `AuthorizeAttribute` in the controllers like this:

```csharp
    [RoutePrefix("api/v1/customers")]
    [Restrict(ClaimTypes.Customers, ClaimValues.FullAccess | ClaimValues.ReadOnly)]
    public class CustomersController
    {
      //...
    }
```

In the example above the CustomersController could be accessed by a user that has the Customer Claim Type with FullAccess or ReadOnly.  The custom attribute also allows the user to pass overriding claims at the method level.

Our custom attribute looks something like this:

```csharp

namespace API.Claims
{
    /// <summary>
    ///     Retricts a controller or method based on ClaimType and OR-ed ClaimValue bit flags.
    ///     See the enum ClaimValue for possible values.
    ///     Will return 401 Unauthorized if user does not have proper claim in token.
    /// </summary>
    [AttributeUsage(AttributeTargets.Method | AttributeTargets.Class, AllowMultiple = true)]
    public class RestrictAttribute : AuthorizeAttribute
    {
        private readonly int _claimType;
        private readonly ClaimValues _claimValues;
        private ClaimValues _fullAccess;


        /// <summary>
        /// Adds Claim and Values to be compared against the OwinContext
        /// </summary>
        /// <param name="ct"></param>
        /// <param name="cv"></param>
        public RestrictAttribute(int ct, ClaimValues cv)
        {
            _claimType = ct;
            _claimValues = cv;
        }

        /// <summary>
        /// Passes the FullAccess claim value as a bypass
        /// </summary>
        /// <param name="fullAccess"></param>
        public RestrictAttribute(ClaimValues fullAccess)
        {
            this._fullAccess = fullAccess;
        }

        private static ClaimTypeValue[] GetClaimTypeValuesFromOwinContext(HttpActionContext actionContext)
        {
            OwinContext ctx = actionContext.Request.Properties["MS_OwinContext"] as OwinContext;
            ClaimTypeValue[] claims = ctx?.Get<ClaimTypeValue[]>(OwinKeys.ClaimValues);
            return claims;
        }

        /// <summary>
        /// Determines if the user can access the given route
        /// </summary>
        /// <param name="actionContext"></param>
        /// <returns></returns>
        protected override bool IsAuthorized(HttpActionContext actionContext)
        {
            if (!base.IsAuthorized(actionContext))
            {
                return false;
            }

            // get claims of user (through token)
            ClaimsPrincipal user = (ClaimsPrincipal) actionContext.ControllerContext.RequestContext.Principal;

            // check claims
            return _claimValues == 0 || CheckClaim(GetClaimTypeValuesFromOwinContext(actionContext), _claimType,
                       _claimValues);
        }

        /// <summary>
        /// Checks if the Claim value exist in the owin claims
        /// </summary>
        /// <param name="claims"></param>
        /// <param name="ct"></param>
        /// <param name="cv"></param>
        /// <returns></returns>
        public static bool CheckClaim(ClaimTypeValue[] claims, int ct, ClaimValues cv)
        {
            if (cv.HasFlag(ClaimValues.All))
            {
                return false;
            }
            // pulling out the ClaimValue that matches the type
            ClaimTypeValue claim = claims.SingleOrDefault(c => c.ClaimTypeId == ct);
            // if none are found, return false for refuse access
            if (claim == null)
            {
                return false;
            }
            // if ReadOnly, then return true, because we have at least that access
            if (cv.HasFlag(ClaimValues.ReadOnly))
            {
                return true;
            }
            // else we need to make sure that we have full access
            return (int) ClaimValues.FullAccess == claim.ClaimValueId;

        }

    }
}
```
One important thing to note here is that we are using OWIN for our API.  I don't plan to cover anything directly related to OWIN, but we did
